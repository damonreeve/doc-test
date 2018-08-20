<!----- Conversion time: 2.152 seconds.


Using this Markdown file:

1. Cut and paste this output into your source file.
2. See the notes and action items below regarding this conversion run.
3. Check the rendered output (headings, lists, code blocks, tables) for proper
   formatting and use a linkchecker before you publish this page.

Conversion notes:

* GD2md-html version 1.0β11
* Sun Aug 19 2018 23:10:32 GMT-0700 (PDT)
* Source doc: https://docs.google.com/a/hereford.tech/open?id=1Mcya2lKF6QqyRjOzM6O-C34LHOKRKd2ERirHJJrQUn8
* This document has images: check for >>>>>  gd2md-html alert:  inline image link in generated source and store images to your server.
----->


<p style="color: red; font-weight: bold">>>>>>  gd2md-html alert:  ERRORs: 0; WARNINGs: 0; ALERTS: 1.</p>
<ul style="color: red; font-weight: bold"><li>See top comment block for details on ERRORs and WARNINGs. <li>In the converted Markdown or HTML, search for inline alerts that start with >>>>>  gd2md-html alert:  for specific instances that need correction.</ul>

<p style="color: red; font-weight: bold">Links to alert messages:</p><a href="#gdcalert1">alert1</a>

<p style="color: red; font-weight: bold">>>>>> PLEASE check and correct alert issues and delete this message and the inline alerts.<hr></p>


<h2>Ozone Project Prebid Server Publisher Onboarding Documentation</h2>


Version


<table>
  <tr>
   <td>VERSION
   </td>
   <td>DATE
   </td>
   <td>AUTHOR
   </td>
  </tr>
  <tr>
   <td>2.0
   </td>
   <td>23/04/2018
   </td>
   <td>Rupesh Lakhani (<a href="mailto:rupesh@askrupert.net">rupesh@askrupert.net</a>)
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
</table>


This document explains how to integrate with the Ozone Project Prebid Server by modifying your current prebid.js client-side integration.


```
No change to existing demand partner configuration: The changes outlined here are to add an additional adaptor. There is no direct impact or modifications to your current demand partner setup and configurations. View the Integration Dataflow Diagram for more info.

New ids will be sourced from demand partners and these will configured in Prebid Server.
```


<h2>Summary of changes</h2>


The following changes will be required in order to integrate with Ozone Project Prebid Server:



1.  Add the Prebid Server Adaptor to your prebid.js build
1.  Add Ozone Project S2S Configuration values to your Prebid Server Adaptor config
1.  Add Lotame and Custom Data variables
1.  Wrap pbjs.que.push function in a timeout to assist loading Lotame data (Optional)

    ```
Note: If your current version of prebid.js is older than v.1.0 you'll need to amend your code because some methods have been deprecated and this integration requires you to be on a version of prebid 1.0 or higher. See Appendix A for more details.
```



<h4>Reference Implementation</h4>


To support this document we have created a simple reference implementation of this code here:

[http://demo.the-ozone-project.com/](http://demo.the-ozone-project.com/) 

<h3>1. Add Ozone Project adaptor to your prebid.js build</h3>


Rebuild your prebid.js file to also include the "Prebid Server" adapter (in addition to your existing client adaptors). The prebid.org site allows you to generate a build from here - [http://prebid.org/download.html](http://prebid.org/download.html) 

Alternatively your engineering team can compile themselves by downloading the prebid server adaptor from here:

[https://github.com/prebid/Prebid.js/blob/master/modules/prebidServerBidAdapter.js](https://github.com/prebid/Prebid.js/blob/master/modules/prebidServerBidAdapter.js)

<h3>2. Add Ozone Project S2S Configuration</h3>


Add the Ozone Project s2sConfig object to your prebid.js configuration.


```
pbjs.setConfig({  
    debug: true,  
    priceGranularity: customGranularity,  //optional
    enableSendAllBids: true, //optional
    s2sConfig: {  
        accountId: '1',  
        enabled: true,  
        bidders: ['appnexus','openx'], 
        timeout: 1000,  
        adapter: 'prebidServer',  
        is_debug: 'true',  
        endpoint: 'http://elb.the-ozone-project.com/openrtb2/auction',  
        syncEndpoint: 'https://prebid.adnxs.com/pbs/v1/cookie_sync',
        cookieSet: true,
        cookiesetUrl: 'https://acdn.adnxs.com/cookieset/cs.js'  
      },
      userSync: {
  iframeEnabled: true
      }
});  
```


More details on how this is done can be found on the prebid.js site: [http://prebid.org/dev-docs/bidders/prebidServer.html](http://prebid.org/dev-docs/bidders#prebidServer). (please ensure you use the prebid server config values below and ignore the steps around signing up for an account with appnexus).


```
NOTE: priceGranularity is only required if you currently utilise the Prebid.js PriceGranularity function and likewise enableSendAllBids.
```


<h3>3. Add Lotame & Custom Data variables</h3>


You will need to amend any prebid.js bidder objects in your prebid.js configuration to contain any custom data and to include a variable to define/use Lotame data in ad requests.

In this example, we've extended the 'appnexus' bidder to contain two additional attributes **lotame **and **customData**. These should be added to have your respective Lotame JSON response and any custom data passed to the respective attributes. You will want to add these to each bidder you defined in the s2sConfig.bidders array in Step 2.

Please refer to Appendix B for confirmation of the format the JSON you pass to both should adhere too.


<table>
  <tr>
   <td><code>var adUnits = [{   \
    code: 'div-gpt-ad-1460505748561-0',   \
    mediaTypes: {   \
        banner: {   \
            sizes: [   \
                [300, 250],   \
                [300, 600]   \
            ]   \
        }   \
    },   \
    bids: [{</code>
   </td>
  </tr>
  <tr>
   <td><code>        bidder: 'appnexus',   \
        params: {   \
            placementId: '[PLACEMENT ID]',   \
          <strong>  lotame: pbLotameData,   \
            customData: pbCustomData  </strong> \
        }  </code>
   </td>
  </tr>
  <tr>
   <td><code>    }]   \
 } \
}];</code>
   </td>
  </tr>
</table>


<h3>4. Wrap pbjs.que.push function inside a timeout (Optional)</h3>


You may find some issues with synchronising your lotame response with the Prebid Server requests. (I.E lotame values are returned to the client after your page has made its prebid.js requests).

The standard pbjs.que.push(function() …. ) code can be delayed to allow for your lotame response to come back before the prebid.js process starts. 

This is done by moving it inside of a setTimeout() with a timeout set to a few hundred milliseconds. 

The existing code can generally be simply picked up & placed in the function body in setTimeout(function(){ …… }, nnn).


```
setTimeout(function() {  
     pbjs.que.push(function() {…  
        the usual code you use to request bids…  
    });  
}, 700);
```


<h3></h3>


<h3>Appendix A - Legacy prebid.js code methods that may require updating</h3>


If you are regenerating your prebid.js file to include the Prebid Server adapter you might find that you have code that now calls pbjs methods that no longer exist 

Two common methods that you maybe currently using that require updating are:



1.  pbjs.setPriceGranularity(…) – This method no longer exists so this needs adding to pbjs.SetConfig. Note that customGranularity contains the same object we were previously sending to the setPriceGranularity(…) method. 
1.  pbjs.enableSendAllBids() – This method no longer exists so as per setPriceGranularity this will need to be added to pbjs.SetConfig

You can use the following links for more details on how these methods have changed and how to update them: \


[http://prebid.org/dev-docs/publisher-api-reference.html#module_pbjs.setPriceGranularity](http://prebid.org/dev-docs/publisher-api-reference.html#module_pbjs.setPriceGranularity)

[http://prebid.org/dev-docs/publisher-api-reference.html#module_pbjs.enableSendAllBids](http://prebid.org/dev-docs/publisher-api-reference.html#module_pbjs.enableSendAllBids) 

<h3></h3>


<h3>Appendix B - Structure of Custom Data and Lotame Data</h3>


The lotame attribute should have a JSON object passed to it which should adhere to the format below:


```
var pbLotameData = {
	"Profile": {
		"tpid": "c8ef27a0d4ba771a81159f0d2e792db4",
		"Audiences": {
			"Audience": [{
				"id": "99999",
				"abbr": "sports"
			}, {
				"id": "88888",
				"abbr": "movie"
			}, {
				"id": "77777",
				"abbr": "blogger"
			}],
			"ThirdPartyAudience": [{
				"id": "123",
				"name": "Automobiles"
			}, {
				"id": "456",
				"name": "Ages: 30-39"
			}]
		}
	}
}
```


 \


The customData attribute should have a JSON object passed to it which should adhere to the format below:


```
var pbcustomData =  JSON.stringify({  
    "topics": "world cup",  
    "kw": "politics, sport, news",  
    "aid": "32432",  
    "search": "null",  
    "article_type": "video"
}); 
```


 \


<h3>Appendix C - PREBID.JS T PREBID SERVER DATAFLOW</h3>


The diagram below describes the addition of Prebid Server S2S adapter to a prebid.js configuration and corresponding dataflow for demand partner placement ids.



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/Ozone-Project0.png). Store image on your image server and adjust path/filename if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/Ozone-Project0.png "image_tooltip")



<!-- GD2md-html version 1.0β11 -->

OSPF Fundamentals
=================


OSPF Introduction
-----------------


Open Shortest Path First (OSPF) is a dynamic link-state routing protocol designed for routers to share Link State DataBases (LSDB) which contain sets of all available routes. The Shortest Past First (SPF) algorithm then calculates the ideal route between two endpoints depending on link state (up or down) and “cost” which is determined by bandwidth.

I used the following topology to study the different aspects of OSPF.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/ospf/multi-area-ospf.png)

OSPF Routers
------------

*   Area Border Router (ABR) is a router that has interfaces in two or more OSPF areas.

*   Internal Router (IR) is a router that has interfaces in only one area.

*   Backbone Router (BR) are routers located in the backbone area.

*   Autonomous System Boundary Router (ASBR) advertises routes from another AS into the OSPF domain.

*   Designated Router (DR) in a broadcast network type all OSPF routers in the same area will establish a DR and BDR to be the only two devices that can listen to LSA broadcasts (new multicast address 224.0.0.6) and the only two devices that will send LSA’s to listen to.

*   Backup Designated Router (BDR) is elected in the event the DR becomes unavailable.

OSPF Neighborship
-----------------

OSPF routers need to establish “neighborship” to exchange routes.  OSPF Routers will send “Hello” packets via multicast (224.0.0.5) at a customizable interval. Any other OSPF routers will also be listening for Hello messages and broadcasting their own. When a router sees its own router id in a foreign hello packet neighborship is considered to have begun.

OSPF routers store all learned routes in a Link State Database (LSDB). Any change in the LSDB requires SPF to recalculate the routing table.

MessageTypes:
-------------

*   Hello: “Introduction” packet that is sent when an OSPF router comes online. Hello packets include the origin router ID and known router ID’s (adjacent routers).

*   DBD: The Database Description is an index of known prefixes/routes to determine if two LSDB’s are identical.

*   LSR: Link State Request packets are sent when there is a discrepency in LSDBs found via the DBD. When entries are missing the lacking router will request information about these routes from a knowledgeable router.

*   LSU: Link State Updates are sent as a response to a LSR. LSU’s will contain information about the “missing” route in the requesting routers LSDB.

*   LSAck: Link State Acknowledgements are sent as a response to a LSU. Like any ACK packet its goal is to verify successful RX of the preceding packet.

Link State Advertisements (LSA)
-------------------------------

OSPF routers will advertise networks using a LSA packet. Different LSA’s are used depending on where the advertisement came from and where it is going to. 

You can view an OSPF routers learned LSA’s from privilege  mode:

> \# show ip ospf database

LSA’s will stay in the  LSDB for 36000 seconds (1 hour) until they time out. LSA’s are typically rebroadcast at 18000 seconds (30 minutes).

All LSA’s will have a Link State Sequence Number (shown in the database as ls seq number:). The sequence number grows incrementally every time a new LSA is rebroadcast starting at 80000000. This exists inside the SPF calculation so if SPF is recalculated the database is destroyed and the process starts over.

Common LSA Types
----------------

*   **Type 1**: The “Router LSA” advertises all links directly connected to the router. Type 1 LSA’s will always be contained inside the OSPF area they are in. This is the only LSA that can advertise multiple networks with the same LSA.
    *   To see the information that is being sent from the router itself:
        
        > \# show ip ospf database router self-originate
        
    *   Link State _ID_: Router ID
    *   If an OSPF router is broadcasting Hello packets on a link but gets no response/neighborship then the interface is considered a Stub link.
        *   For security practices it is always a good idea to set an OSPF interface to “Passive” (not accepting incoming messages) if you do not have neighbors on that link.
    *   The Link type will be on Transit Links or Stub Links.
        *   In a transit network the packets traveling the link may have destination and source IP’s that are not related to the subnet configured on the link itself.
            *   Transit links will always have a neighbor and have a network type of Broadcast or Non Broadcast Multi Access.
            *   A Stub link will mean that any traffic on the link is in the same subnet as there are no other ospf routers.
    *   Type 1 LSA’s provide a routers interfaces and DR information but not subnet mask so cannot complete the logical topology.

*   **Type 2**: The “Network LSA” is a DR advertising connected router information though the area they are broadcasted in.
    *   Only the Designated Router will create Network LSA’s. 
    *   From any router on the link:  
        
        > \# show ip ospf database network adv-router
        
    *   Advertising Router: provides the DR ip address
    *   Network Mask will provide the subnet mask as well as any neighbor interface addresses.

*   **Type 3**: The “Summary LSA” is when an ABR translates a type 1 LSA into a type 3 for inter area communication.
    *   To view summary LSA’s:  
        
        > \# show ip ospf database summary
        
    *   Contains information learned from Type 1 and Type 2 LSAs.
        *   Network Address
        *   Subnet Mask
        *   Cost/Metric

*   **Type 4**: The “ASBR Summary LSA” is when an ABR translates type 1 LSA’s from the ASBR into Type 4 LSA’s inside the area.
    *   Advertises the route to an ASBR. 
    *   Broadcasted in all areas that do not have the origin ASBR.
    *   To view:  
        
        > \# show ip ospf database asbr-summary
        
    *   Link State ID: ASBR router id
    *   Adv router: ABR router id

*   **Type 5**: The “ASBR external route LSA” is when a default route or redistributed route is advertised through all areas. Only LSA that is not stopped by ABR’s.
    *   To view External LSA’s:  
        
        > \# show ip ospf database external
        
    *   LSDB does not tag area number on external LSA’s as they are broadcast through all areas.
    *   Link ID: redistributed networks
    *   ADV Router: ASBR router ID
    *   Network Mask:
        *   subnet mask
        *   cost
        *   forwarding address/default route
    *   Does not advertise ASBR interfaces so unrouteable in different areas until combined with ASBR summary LSA>

*   **Type 6**: unused multicast LSA

*   **Type 7**: the “NSSA External LSA” is a route from a Not So Stubby Area before it is translated into LSA 5 at the ABR.
    *   Essentially this is just an External Route LSA but as NSSA’s cannot have LSA 5 inside the area it is considered a different LSA type.

OSPF Areas
----------

An OSPF “Area” is a routing environment where all OSPF routers involved will share type 1 LSA’s and have identical LSDB’s. In my lab environments one area will always be enough but in an enterprise environment where you could have several thousand routes this would involve a lot of control plane overhead potentially maxing out devices and causing drops. Every time a link drops the entire area has to recalculate their SPF and update LSDB causing the same issue to happen over and over. A link that is flapping would blow up mediocre gear in no time. 

To mitigate this you could summarize and filter advertised routes. If every OSPF router has to have an identical LSDB in the same area though it is impossible to do with only one area. Route filtering and summarization occurs at the edge/between areas using the Area Border Router. Depending on the configuration the ABR will stop certain types of LSA’s or routes from being rebroadcast into a neighboring area.

By default ABR’s will:

*   stop router and network LSA’s and retranslate into summary LSAs. 
*   stop foreign summary LSA’s and retranslate them replacing the adv-ip and mask with their own.
*   Stop AS Summary LSA’s.

Backbone Area
-------------

The Backbone Area or Area 0 is the centralized location for the OSPF domain. This is the origin of all the OSPF inter area communication and the nexus to which all other areas are connected.

On all routers in area:

> (config) router ospf x
> 
> network A.B.C.D wildcardmask area 0

x being the instance id of OSPF on that router.

Regular Area
------------

Regular Areas are similar to the backbone area in functionality. Regular areas send and receive LSA’s in the same manner as the backbone area (which is considered a regular area).

On all routers in area:

> (config) router ospf x
> 
> network A.B.C.D. wildcardmask area y

x being the instance id of OSPF on that router and y being the area you would like to establish.

Stubby Area
-----------

Stubby Areas are similar to Regular Area’s except they block all incoming ASBR LSA’s (LSA 4 and LSA 5). LSA 5 is translated to LSA 3 via a default route for use inside the area.

Stub areas cannot have virtual links in them, have an ASBR inside the area or be Area 0.

On all routers in area:

> (config)  area x stub

Totally Stubby Area
-------------------

Totally Stubby Area’s are similar to Stubby Area’s but in addition to blocking LSA 4 and 5 they also block LSA 3. The ABR will send a default route instead of translating LSA 5 to LSA 3. A default route will also be sent for all external type 3 LSA’s (not re translated).

TSA’s cannot have virtual links in them, have an ASBR inside the area or be Area 0.

On ABR:

> (config) area x stub no-summary 

Not So Stubby Area
------------------

Not So Stubby Area (NSSA) are similar to Stubby areas in that they block ASBR Summaries and ASBR External LSA’s. 

NSSA’s cannot have virtual links in them or be Area 0 but can have an ASBR inside the area. The ABR will translate ASBR External LSA’s to Summary LSA’s for incoming traffic and NSSA LSA’s to ASBR External and ASBR summary LSA’s for communication outside the area.

On ABR:

> area x nssa default-information-originate

Totally Not So Stubby Area
--------------------------

Totally Not So Stubby Area (TNSSA) are similar to NSSA in that they block ASBR Eternal and Summary LSA’s but they will also block Summary LSA’s externally and translate all of them to a default route summary LSA. NSSA LSA’s broadcasted within the area will be translated to ASBR external and summary LSA’s to be advertised out through ABR.

On ABR:

> area x nssa no -summary

### Submit a Comment [Cancel reply](/?p=989#respond)

Your email address will not be published. Required fields are marked \*

Comment \*

Name \* 

Email \* 

Website 

 Save my name, email, and website in this browser for the next time I comment.

  

Search for:  

#### Recent Posts

*   [Bare Metal Server to ESXI](https://phantasmagoria.tech/?p=1425)
*   [Virtual ISP – Update 1](https://phantasmagoria.tech/?p=1308)
*   [GPON Fundamentals](https://phantasmagoria.tech/?p=1257)
*   [Kubernetes – High Level](https://phantasmagoria.tech/?p=1289)
*   [EIGRP Fundamentals](https://phantasmagoria.tech/?p=1070)

#### Recent Comments

*   [Tavish](https://phantasmagoria.tech) on [First OSPF Lab](https://phantasmagoria.tech/?p=1#comment-1)

#### Archives

*   [December 2021](https://phantasmagoria.tech/?m=202112)
*   [November 2021](https://phantasmagoria.tech/?m=202111)
*   [August 2021](https://phantasmagoria.tech/?m=202108)
*   [July 2021](https://phantasmagoria.tech/?m=202107)

#### Categories

*   [Documentation](https://phantasmagoria.tech/?cat=10)
*   [EIGRP](https://phantasmagoria.tech/?cat=14)
*   [Networking](https://phantasmagoria.tech/?cat=4)
*   [OSPF](https://phantasmagoria.tech/?cat=6)
*   [Routing](https://phantasmagoria.tech/?cat=5)
*   [Soft Skills](https://phantasmagoria.tech/?cat=13)
*   [Switching](https://phantasmagoria.tech/?cat=7)
*   [Tutorial](https://phantasmagoria.tech/?cat=12)
*   [Uncategorized](https://phantasmagoria.tech/?cat=1)
*   [Work in Progress](https://phantasmagoria.tech/?cat=11)

#### Meta

*   [Log in](https://phantasmagoria.tech/wp-login.php)
*   [Entries feed](https://phantasmagoria.tech/?feed=rss2)
*   [Comments feed](https://phantasmagoria.tech/?feed=comments-rss2)
*   [WordPress.org](https://wordpress.org/)

(function() { var file = \["https:\\/\\/phantasmagoria.tech\\/wp-content\\/et-cache\\/989\\/et-divi-dynamic-989-late.css"\]; var handle = document.getElementById('divi-style-inline-inline-css'); var location = handle.parentNode; if (0===document.querySelectorAll('link\[href="' + file + '"\]').length) { var link = document.createElement('link'); link.rel = 'stylesheet'; link.id = 'et-dynamic-late-css'; link.href = file; location.insertBefore(link, handle.nextSibling); } })(); var et\_animation\_data = \[{"class":"et\_pb\_blurb\_10000","style":"slideLeft","repeat":"once","duration":"1000ms","delay":"0ms","intensity":"5%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_button\_10000","style":"slideLeft","repeat":"once","duration":"1000ms","delay":"0ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_button\_10001","style":"slideLeft","repeat":"once","duration":"1000ms","delay":"0ms","intensity":"70%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_10001","style":"fold","repeat":"once","duration":"1000ms","delay":"0ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_10002","style":"fold","repeat":"once","duration":"1000ms","delay":"150ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_10003","style":"fold","repeat":"once","duration":"1000ms","delay":"200ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_10004","style":"fold","repeat":"once","duration":"1000ms","delay":"450ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_10005","style":"fold","repeat":"once","duration":"1000ms","delay":"100ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_10006","style":"fold","repeat":"once","duration":"1000ms","delay":"300ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_testimonial\_10000","style":"fold","repeat":"once","duration":"1000ms","delay":"0ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_testimonial\_10001","style":"fold","repeat":"once","duration":"1000ms","delay":"400ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_testimonial\_10002","style":"fold","repeat":"once","duration":"1000ms","delay":"200ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_testimonial\_10003","style":"fold","repeat":"once","duration":"1000ms","delay":"150ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_testimonial\_10004","style":"fold","repeat":"once","duration":"1000ms","delay":"400ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_testimonial\_10005","style":"fold","repeat":"once","duration":"1000ms","delay":"400ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_testimonial\_10006","style":"fold","repeat":"once","duration":"1000ms","delay":"400ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_testimonial\_10007","style":"fold","repeat":"once","duration":"1000ms","delay":"400ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_0","style":"fold","repeat":"once","duration":"1000ms","delay":"150ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_1","style":"fold","repeat":"once","duration":"1000ms","delay":"150ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"},{"class":"et\_pb\_blurb\_2","style":"fold","repeat":"once","duration":"1000ms","delay":"150ms","intensity":"50%","starting\_opacity":"0%","speed\_curve":"ease-in-out"}\]; /\* <!\[CDATA\[ \*/ jqueryParams.length&&$.each(jqueryParams,function(e,r){if("function"==typeof r){var n=String(r);n.replace("$","jQuery");var a=new Function("return "+n)();$(document).ready(a)}}); /\* \]\]> \*/ /\* <!\[CDATA\[ \*/ var DIVI = {"item\_count":"%d Item","items\_count":"%d Items"}; var et\_builder\_utils\_params = {"condition":{"diviTheme":true,"extraTheme":false},"scrollLocations":\["app","top"\],"builderScrollLocations":{"desktop":"app","tablet":"app","phone":"app"},"onloadScrollLocation":"app","builderType":"fe"}; var et\_frontend\_scripts = {"builderCssContainerPrefix":"#et-boc","builderCssLayoutPrefix":"#et-boc .et-l"}; var et\_pb\_custom = {"ajaxurl":"https:\\/\\/phantasmagoria.tech\\/wp-admin\\/admin-ajax.php","images\_uri":"https:\\/\\/phantasmagoria.tech\\/wp-content\\/themes\\/Divi\\/images","builder\_images\_uri":"https:\\/\\/phantasmagoria.tech\\/wp-content\\/themes\\/Divi\\/includes\\/builder\\/images","et\_frontend\_nonce":"5c1443d287","subscription\_failed":"Please, check the fields below to make sure you entered the correct information.","et\_ab\_log\_nonce":"7bb8ac249f","fill\_message":"Please, fill in the following fields:","contact\_error\_message":"Please, fix the following errors:","invalid":"Invalid email","captcha":"Captcha","prev":"Prev","previous":"Previous","next":"Next","wrong\_captcha":"You entered the wrong number in captcha.","wrong\_checkbox":"Checkbox","ignore\_waypoints":"no","is\_divi\_theme\_used":"1","widget\_search\_selector":".widget\_search","ab\_tests":\[\],"is\_ab\_testing\_active":"","page\_id":"989","unique\_test\_id":"","ab\_bounce\_rate":"5","is\_cache\_plugin\_active":"no","is\_shortcode\_tracking":"","tinymce\_uri":""}; var et\_pb\_box\_shadow\_elements = \[\]; /\* \]\]> \*/ /\* <!\[CDATA\[ \*/ var localize = {"ajaxurl":"https:\\/\\/phantasmagoria.tech\\/wp-admin\\/admin-ajax.php","nonce":"324c12a8ab","i18n":{"added":"Added ","compare":"Compare","loading":"Loading..."},"eael\_translate\_text":{"required\_text":"is a required field","invalid\_text":"Invalid","billing\_text":"Billing","shipping\_text":"Shipping","fg\_mfp\_counter\_text":"of"},"page\_permalink":"https:\\/\\/phantasmagoria.tech\\/?p=989","cart\_redirectition":"","cart\_page\_url":"","el\_breakpoints":{"mobile":{"label":"Mobile Portrait","value":767,"default\_value":767,"direction":"max","is\_enabled":true},"mobile\_extra":{"label":"Mobile Landscape","value":880,"default\_value":880,"direction":"max","is\_enabled":false},"tablet":{"label":"Tablet Portrait","value":1024,"default\_value":1024,"direction":"max","is\_enabled":true},"tablet\_extra":{"label":"Tablet Landscape","value":1200,"default\_value":1200,"direction":"max","is\_enabled":false},"laptop":{"label":"Laptop","value":1366,"default\_value":1366,"direction":"max","is\_enabled":false},"widescreen":{"label":"Widescreen","value":2400,"default\_value":2400,"direction":"min","is\_enabled":false}}}; /\* \]\]> \*/ /\* <!\[CDATA\[ \*/ var mejsL10n = {"language":"en","strings":{"mejs.download-file":"Download File","mejs.install-flash":"You are using a browser that does not have Flash player enabled or installed. Please turn on your Flash player plugin or download the latest version from https:\\/\\/get.adobe.com\\/flashplayer\\/","mejs.fullscreen":"Fullscreen","mejs.play":"Play","mejs.pause":"Pause","mejs.time-slider":"Time Slider","mejs.time-help-text":"Use Left\\/Right Arrow keys to advance one second, Up\\/Down arrows to advance ten seconds.","mejs.live-broadcast":"Live Broadcast","mejs.volume-help-text":"Use Up\\/Down Arrow keys to increase or decrease volume.","mejs.unmute":"Unmute","mejs.mute":"Mute","mejs.volume-slider":"Volume Slider","mejs.video-player":"Video Player","mejs.audio-player":"Audio Player","mejs.captions-subtitles":"Captions\\/Subtitles","mejs.captions-chapters":"Chapters","mejs.none":"None","mejs.afrikaans":"Afrikaans","mejs.albanian":"Albanian","mejs.arabic":"Arabic","mejs.belarusian":"Belarusian","mejs.bulgarian":"Bulgarian","mejs.catalan":"Catalan","mejs.chinese":"Chinese","mejs.chinese-simplified":"Chinese (Simplified)","mejs.chinese-traditional":"Chinese (Traditional)","mejs.croatian":"Croatian","mejs.czech":"Czech","mejs.danish":"Danish","mejs.dutch":"Dutch","mejs.english":"English","mejs.estonian":"Estonian","mejs.filipino":"Filipino","mejs.finnish":"Finnish","mejs.french":"French","mejs.galician":"Galician","mejs.german":"German","mejs.greek":"Greek","mejs.haitian-creole":"Haitian Creole","mejs.hebrew":"Hebrew","mejs.hindi":"Hindi","mejs.hungarian":"Hungarian","mejs.icelandic":"Icelandic","mejs.indonesian":"Indonesian","mejs.irish":"Irish","mejs.italian":"Italian","mejs.japanese":"Japanese","mejs.korean":"Korean","mejs.latvian":"Latvian","mejs.lithuanian":"Lithuanian","mejs.macedonian":"Macedonian","mejs.malay":"Malay","mejs.maltese":"Maltese","mejs.norwegian":"Norwegian","mejs.persian":"Persian","mejs.polish":"Polish","mejs.portuguese":"Portuguese","mejs.romanian":"Romanian","mejs.russian":"Russian","mejs.serbian":"Serbian","mejs.slovak":"Slovak","mejs.slovenian":"Slovenian","mejs.spanish":"Spanish","mejs.swahili":"Swahili","mejs.swedish":"Swedish","mejs.tagalog":"Tagalog","mejs.thai":"Thai","mejs.turkish":"Turkish","mejs.ukrainian":"Ukrainian","mejs.vietnamese":"Vietnamese","mejs.welsh":"Welsh","mejs.yiddish":"Yiddish"}}; /\* \]\]> \*/ /\* <!\[CDATA\[ \*/ var \_wpmejsSettings = {"pluginPath":"\\/wp-includes\\/js\\/mediaelement\\/","classPrefix":"mejs-","stretching":"responsive","audioShortcodeLibrary":"mediaelement","videoShortcodeLibrary":"mediaelement"}; /\* \]\]> \*/ /\* <!\[CDATA\[ \*/ var elementorFrontendConfig = {"environmentMode":{"edit":false,"wpPreview":false,"isScriptDebug":false},"i18n":{"shareOnFacebook":"Share on Facebook","shareOnTwitter":"Share on Twitter","pinIt":"Pin it","download":"Download","downloadImage":"Download image","fullscreen":"Fullscreen","zoom":"Zoom","share":"Share","playVideo":"Play Video","previous":"Previous","next":"Next","close":"Close","a11yCarouselWrapperAriaLabel":"Carousel | Horizontal scrolling: Arrow Left & Right","a11yCarouselPrevSlideMessage":"Previous slide","a11yCarouselNextSlideMessage":"Next slide","a11yCarouselFirstSlideMessage":"This is the first slide","a11yCarouselLastSlideMessage":"This is the last slide","a11yCarouselPaginationBulletMessage":"Go to slide"},"is\_rtl":false,"breakpoints":{"xs":0,"sm":480,"md":768,"lg":1025,"xl":1440,"xxl":1600},"responsive":{"breakpoints":{"mobile":{"label":"Mobile Portrait","value":767,"default\_value":767,"direction":"max","is\_enabled":true},"mobile\_extra":{"label":"Mobile Landscape","value":880,"default\_value":880,"direction":"max","is\_enabled":false},"tablet":{"label":"Tablet Portrait","value":1024,"default\_value":1024,"direction":"max","is\_enabled":true},"tablet\_extra":{"label":"Tablet Landscape","value":1200,"default\_value":1200,"direction":"max","is\_enabled":false},"laptop":{"label":"Laptop","value":1366,"default\_value":1366,"direction":"max","is\_enabled":false},"widescreen":{"label":"Widescreen","value":2400,"default\_value":2400,"direction":"min","is\_enabled":false}}}, "version":"3.19.2","is\_static":false,"experimentalFeatures":{"e\_optimized\_assets\_loading":true,"e\_optimized\_css\_loading":true,"additional\_custom\_breakpoints":true,"block\_editor\_assets\_optimize":true,"ai-layout":true,"landing-pages":true,"e\_image\_loading\_optimization":true,"e\_global\_styleguide":true},"urls":{"assets":"https:\\/\\/phantasmagoria.tech\\/wp-content\\/plugins\\/elementor\\/assets\\/"},"swiperClass":"swiper-container","settings":{"page":\[\],"editorPreferences":\[\]},"kit":{"active\_breakpoints":\["viewport\_mobile","viewport\_tablet"\],"global\_image\_lightbox":"yes","lightbox\_enable\_counter":"yes","lightbox\_enable\_fullscreen":"yes","lightbox\_enable\_zoom":"yes","lightbox\_enable\_share":"yes","lightbox\_title\_src":"title","lightbox\_description\_src":"description"},"post":{"id":989,"title":"OSPF%20Fundamentals%20-%20Phantasmagoria","excerpt":"","featuredImage":false}}; /\* \]\]> \*/ /\* <!\[CDATA\[ \*/ var \_wpUtilSettings = {"ajax":{"url":"\\/wp-admin\\/admin-ajax.php"}}; /\* \]\]> \*/ /\* <!\[CDATA\[ \*/ var wpformsElementorVars = {"captcha\_provider":"recaptcha","recaptcha\_type":"v2"}; /\* \]\]> \*/

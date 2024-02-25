# TYPO3 v12.4 - Leaflet - OpenStreetMap - Content Security Policy

Creation of a simple map with Leaflet and OpenStreetMap for the TYPO3 frontend, taking into account the content security policy (CSP). The aim is to avoid inline scripts.

TYPO3: Based on [TYPO3 Documentation CSP](https://docs.typo3.org/m/typo3/reference-coreapi/main/en-us/ApiOverview/ContentSecurityPolicy/) Many Thanks!

Leaflet: Based on [Leaflet Tutorials](https://leafletjs.com/examples.html) Many Thanks!
 
## TYPO3 settings
The feature flag security.frontend.enforceContentSecurityPolicy must be activated so that the TYPO3 frontend takes over the handling of the content security policy.
```
Settings > Feature Toggles Global Configuration > Security: frontend enforce content security policy
```
## TYPO3 Content Security Policy
In this example, a PHP file in the Site Packages Extension is used to configure the CSP. The advantage is that the CSP is automatically transferred when the Site Packages Extension is deployed.
The default settings of the MutationCollection ("default-src 'self';") are extended to allow the loading of image files from openstreetmap.de.

EXT:my_site_package/Configuration/ContentSecurityPolicies.php

```
<?php

declare(strict_types=1);

use TYPO3\CMS\Core\Security\ContentSecurityPolicy\Directive;
use TYPO3\CMS\Core\Security\ContentSecurityPolicy\Mutation;
use TYPO3\CMS\Core\Security\ContentSecurityPolicy\MutationCollection;
use TYPO3\CMS\Core\Security\ContentSecurityPolicy\MutationMode;
use TYPO3\CMS\Core\Security\ContentSecurityPolicy\Scope;
use TYPO3\CMS\Core\Security\ContentSecurityPolicy\SourceKeyword;
use TYPO3\CMS\Core\Security\ContentSecurityPolicy\SourceScheme;
use TYPO3\CMS\Core\Security\ContentSecurityPolicy\UriValue;
use TYPO3\CMS\Core\Type\Map;

return Map::fromEntries([
    // Provide declarations for the frontend
    Scope::frontend(),
    // NOTICE: When using `MutationMode::Set` existing declarations will be overridden
    // Code block for CSP configuration
    new MutationCollection(
    // Settings for entire sources. Results in `default-src 'self'`
        new Mutation(
            MutationMode::Set,
            Directive::DefaultSrc,
            SourceKeyword::self
        ),
        new Mutation(
            MutationMode::Extend,
            Directive::ImgSrc,
            SourceScheme::data,
            new UriValue('https://*.openstreetmap.de')
        ),
    ),
]);
```
## Leaflet files
my_site_package/Configuration/TypoScript/setup.typoscript
```
page.includeCSS.map = EXT:my_site_package/Resources/Public/Contrib/Leaflet/leaflet.css
page.includeJS.map = EXT:my_site_package/Resources/Public/Contrib/Leaflet/leaflet.js
```
## Leaflet example map
The settings of the Leaflet map are written to a script file of the site package. The map is loaded if the corresponding Id is available on a page.

my_site_package/Resources/Public/JavaScript/Dist/scripts.js
```
if(document.getElementById('munich')) {
    var map = L.map('map',{
        center: [48.139722, 11.574444],
        dragging: false,
        tap: false,
        zoom: 9
    });

    var marker1 = L.marker([48.139722, 11.574444],{alt: 'Munich'}).addTo(map);
    marker1.bindPopup("Munich, Bavaria, Germany").openPopup();

    L.tileLayer('https://{s}.tile.openstreetmap.de/tiles/osmde/{z}/{x}/{y}.png', {
        'attribution': '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> Contributors', 'useCache': true
    }).addTo(map);
}
```
## TYPO3 include Leaflet map
TYPO3 will load the file with a nonce: <script src="/_assets/8d63...2252/JavaScript/Dist/scripts.js?170...617" nonce="XPKRD3m...Sn2q0Ik9atw"></script>

my_site_package/Configuration/TypoScript/setup.typoscript
```
page.includeJS.my_site_package_scripts = EXT:my_site_package/Resources/Public/JavaScript/Dist/scripts.js
```
## TYPO3 example Content Element HTML
```
<div id="map"></div>
<div id="munich"></div>
```

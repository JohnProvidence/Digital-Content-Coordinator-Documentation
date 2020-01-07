## ProvLibDigital Theme Information

ProvLibDigital (**PLD**) is built on the Islandora Respository framework and uses [Docker](https://www.docker.com/) as an application deployment tool.



## Drupal Specific Information ##
#### :file_folder: PLD Site theme

**Theme Directory:** `pld/sites/all/themes/pld`

The `pld` directory is a [bind mount](https://docs.docker.com/storage/bind-mounts/) to the `var/www/html` directory in the docker container for Apache. You can move files into this folder, modify files, interact with it and it will sync to the directory in the docker container.

The theme `pld` is based on a bare bones Drupal starter theme called [Basic](https://www.drupal.org/project/basic/releases/7.x-4.3).

## Islandora Overrides

**Preprocess Function overrides** `pld/sites/all/themes/pld/includes/`

This directory contains files that override template preprocess functions in various islandora modules. You would override a preprocess function to introduce data you want to pass to the template that is not included in the original preprocess function.  If a module use a preprocess function to generate data that is passed to a template you can override it by including in your theme template.php file the preprocess function you want to override.  In the case of our theme I have set up a directory with a file for each islandora content model/module I'm overriding. Those files are included in the template file using the `require();` function.

:file_folder: **includes directory structure**
* **islandora_basic_image.php** - overrides template functions found in the **islandora solution pack basic image** module.
* **islandora_book.php** - overrides template functions **islandora solution pack book**
* **islandora_bookmark.php** - overrides the template functions for the **islandora bookmark module**
* **islandora_breadcrumb** - overrides the template functions for the **islandora_breadcrumb module**
* **islandora_collections.php** - this used to overrides the template functions for how collections are displayed in our theme. We have switched to using SOLR to generate our collection and search results. Those overrides are found in the module **PLD_Collections**.
* **islandora_compound** - overrides the template functions for the **islandora solution pack compound object modules**
* **islandora_pdf.php** - overrides the template functions for the **islandora solution pack pdf module**
* **islandora_remote_media.php** - overrides the template functions for the **islandora remote media module**
* **islandora_solr_search.php** - some overrides for the template for solr search results
* **islandora_web_archive** - overrides for **islandora solution pack web archive**. This is not complete and will need to be finished.

---

:file_folder: **Template Directory Location:** `pld/sites/all/themes/pld/templates/`

There are a number of files in this directory named `islandora-[some-template-name].tpl.php`. These are the specific files I used to override the presentation of specific parts of islandora and, in some cases, the data being passes to those templates. The biggest modification is the use of larger image datastreams for collection template files and search template files. In most cases I'm calling the **MEDIUM_SIZE** datastream, which is generated on ingest for Basic Image content model and Compound Content model (parent object only). For the Book Content Model I'm using the **JP2** datastream.
I did this to provide larger image thumbnails in grids since the default Islandora thumbnail size is pretty tiny. There may be a better way to do this such as modifying the Islandora module to adjust the size thumbnail datastream produced.

#### Islandora Templates in PLD Theme ####

##### islandora-basic-collection-grid.tpl.php #####

This file overrides the default collection grid view. Our theme currently uses SOLR to generate collection grids as well as grids of returned search results.  Please see **islandora-solr-grid.tpl.php**.

##### islandora-basic-collection-wrapper.tpl.php #####

File that generates the structure of a view for any object with the basic collection content model (all collections, root collection).

###### Modifications Made To This Template: ######

_(lines 24 - 29)_ -

```php
<?php
    if(!empty($dc_array['dc:title']['value'])): ?>

      <?php if($dc_array['dc:title']['value'] == 'Top-level Collection'): ?>

        <h2>Browse Collections</h2>

      <?php else: ?>

       <h2><?php print $dc_array['dc:title']['value']; ?></h2>

    <?php endif; ?>
```

I added a conditional to check if title of the current collection is "Top-Level Collection". If so display the `<h2>Browse Collections</h2>` instead.

_(lines 34 - 37)_ -


```php
<?php  if(!$display_metadata && !empty($dc_array['dc:description']['value'])): ?>
    <p><?php print nl2br($dc_array['dc:description']['value']); ?></p>
<?php endif; ?>
```
I added a check to see if the variable `$display_metadata` is not present and if the variable `$dc_array['dc:description']['value']` is not empty. If so, print the description.

##### islandora-basic-collection.tpl.php #####

This isn't currently being used on our theme. No need to spend a lot of time on this one...

##### islandora-basic-image.tpl.php #####

This outputs the display of any object with the basic image content model.

###### Modifications Made To This Template: ######

_(lines 14 - 18)_

```php
<div class="social-sharing-buttons">
      <div class="social-title">Share This Object:</div>
      <?php print $variables['facebook_button']; ?>
      <?php print $variables['twitter_button']; ?>
</div>
```
**Social Media Sharing** - These two variables pass dynamically generated links to share the content on facebook or twitter.

_(lines 25 - 36)_

```php
<?php if (isset($islandora_content) && $variables['under_copyright'] == NULL): ?>
    <div class="islandora-basic-image-content">
      <?php print $islandora_content; ?>
    </div>
<?php endif; ?>

<?php if($variables['under_copyright'] != NULL): ?>
      <div class="copyright-restriction__message">
        <?php print $variables['under_copyright']; ?>
      </div>
    <?php endif; ?>
```
This checks to make sure the variable `$islandora_content` is set and also checks to make sure the `COPYRIGHT` datastream is NULL. If so the `$islandora_content` variable is displayed.

_(`$islandora_content` = the MEDIUM_SIZE img datastream and the html wrapper)_

If the `COPYRIGHT` datastream is present display a message informing the end user that the image is under copyright restriction.

_(lines 42 - 66)_

```php
<?php if($variables['parent_collection'] != NULL): ?>
    <div class="parent-collection-info__wrapper">
      <h3>In Collection: <a href="<?php print $variables['parent_url']; ?>"><?php print $variables['parent_collection']; ?></a></h3>
      <?php if($variables['collection_finding_aid_button'] != NULL) {
        print $variables['collection_finding_aid_button'];
      }  ?>
    </div>
<?php endif; ?>

<div class="islandora-basic-image-download-btns__wrapper">
<?php if(isset($variables['img_btn']) && $variables['under_copyright'] == NULL): ?>
      <?php print $variables['img_btn']; ?>
<?php endif; ?>
<?php if(isset($variables['mods_btn'])): ?>
      <?php print $variables['mods_btn']; ?>
<?php endif; ?>

<?php if(isset($variables['dc_btn'])): ?>
    <?php print $variables['dc_btn']; ?>
<?php endif; ?>

<?php if(isset($variables['marcxml_btn'])): ?>
    <?php print $variables['marcxml_btn']; ?>
<?php endif; ?>
```

## Finding Aids
---

We display links to Finding Aid datastreams on object display pages.

**Example:** [https://provlibdigital.org/islandora/object/islandora%3A10070](https://provlibdigital.org/islandora/object/islandora%3A10070)

You can expose a finding aid to an object by adding it to the parent collection as a datastream. To do this just add a new datastream to the parent object called **FINDING_AID**.  The file you upload should be the PDF of the collection finding aid.

**Example of a Parent Object** [https://provlibdigital.org/islandora/object/islandora%3A038-03](https://provlibdigital.org/islandora/object/islandora%3A038-03)

## Images Under Copyright Restriction

I also use a datastream to check if an image in our repository is under copyright restriction and, if so, display a message stating that.

**Example** [Under Copyright Example](https://provlibdigital.org/islandora/object/islandora%3A26732?solr_nav%5Bid%5D=070bdc4b5f90ae1fc7ca&solr_nav%5Bpage%5D=1&solr_nav%5Boffset%5D=3)

To set an object to Under Copyright status add a datastream called **COPYRIGHT** to the object in question. The datastream can be anything (an empty text file, for example). I've opted to just make a really simple xml file.

`<copyright>Restricted</copyright>`

It really doesn't matter what's in the file. It could be a .jpg of a Garfield comic or a PDF copy of the Mueller Report. The only thing that matters is that the object has a datastream associated with it called **COPYRIGHT**. The presence of a COPYRIGHT datastream is what the object template is looking for. If that datastream file is present the template will display the copyright message.

## Custom Drupal Modules Built for ProvLibDigital
---

There are three custom drupal modules I've built for the theme. They can all be found locally at `/ISLE/pld/sites/all/modules/islandora/`

:file_folder: **pld_collections**
---

:link: [Github Repo](https://github.com/JohnProvidence/pld_collections)
Overrides the display of collections as generated by SOLR.  Also includes code for the **Recently Added Objects** block that appears on the homepage.

:file_folder: **pld_featured_objects**
---

:link:[Github Repo](https://github.com/JohnProvidence/Islandora-Featured-Collection)

Creates a block at the top of the homepage that pulls in an image from a repository object. The objects choosen have to have a **MEDIUM_SIZE** datastream. Images can be added as a featured object or removed in the **MANAGE** admin screen for each object.

![images/object_manage_screen.png](images/object_manage_screen.png)

_You can add an item by clicking the box in the **Feature This Object In The Featured Collections Region** admin. If the object is already selected a message prompting you to remove it will be displayed.

![images/featured_object_admin.png](images/featured_object_admin.png)

:file_folder: **pld_finding_aids**
---
:link: [Github Repo](https://github.com/JohnProvidence/PLD-Finding-Aids)

This creates a [page that displays all the finding aids in the repository](https://provlibdigital.org/finding-aids). This loops through all repository objects that are members of the root collection (islandora:root) and also have collection as a Content Model. If the object meets that criteria and the object has a datastream **FINDING_AID** then the thumbnail for the collection, the item's description and links to download the finding aid and to view the collection are displayed.

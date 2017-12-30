# Foodsager


# Client Flows and API
 
## General
This document describes the Foodsager application action flow and client-server API.
Foodsager is an application for private recipes management. It will allow the user to:

- Type in, a recipe.
- Scan in, a recipe.
- Search for recipe in own cookbook as well as in the community.
- Import public recipe from other users’ cookbooks.
- Share a recipe with a friend.
- Make her/his recipes publish.
- Advertise products.

> NOTE: This document is a work in progress. There may be some changes in it once the work commence. 


> Foodsager's current client was written with ionic development tool. We are now shifting to native language. Because of this, the implementation of some of the client side will be improved and may require some change in the documentation.  


> Note: Foodsager is already running on many devices. Because of that, if the need arise to change the response to an API call or get a new API call, we will create them in a new endpoint, that will reflect the new version.


> You will have a web page were you can check the functionality of the server. This web page will also show how to make REST calls to the server.

## Multi-Language
Foodsager supports multi language. For that, the client should have a data structure holding all the static text it needs for displaying text. 
The server holds all supported translations of the these texts.


# Client work flows

## Overview
Here is a diagram showing login process to Foodsager:

![Login flow](https://docs.google.com/drawings/d/e/2PACX-1vRx3gU0iFTxNiCSxD9Rd_i8IAEQetym8INjRTIoV_C3UhBWx-ibRnt6K8MYCLX0ErMQMnFOLxyJAubT/pub?w=960&h=720)

## Signing up   
Once the user install the application he needs to register to Foodsager. So the first screen will be the sign-up screen. The user can sign-up to Foodsager’s user account or login via Facebook.
If the user sign up to Foodsager, he will supply an email that will be used as Sign-in name. Then the user type a password. The server will respond with a token that the client will use for any REST call that needs authentication.
On a successful login, the client saves the user name and the password on local storage. On consecutive login attempts, the client should try to login using these credentials.
If the user had logged out, or the password had changed, the client displays the login screen, with the user email filled in. 
If the user sign up via Facebook, the client should ask the server for its [application id](#api-user-fbappid). It will use this to display facebook login screen. Once the user successfully login the client will get a Facebook’s token. The client will send the Facebook (here after also FB) token to Foodsager’s server. The server will validate the FB token and send back a Foodsager. From this point the client uses Foodsager’s token to communicate with the server.

## Log in
User can register via Facebook or Directly via Foodsager.
**Foodsager login**
When the user registers via Foodsager he's credentials and culture code, are stored locally (on the device). 
When the user starts the application, it looks for the saved credentials. If they exists then the application try to login with the credentials (via [/api/user/login](api-user-login)).
If the login fails (not due to network issues) then the application should display the login dialog. The login dialog should have the user's email filled in, and focus on the password.
If there were network issues, the client should switch to offline mode (see relevant section).
On success, the client gets an authentication token from Foodsager. 
On success login the client should save the new email-password culture-code on device, for next time the application starts.
***Forgot password***
If the user cannot remember his password he can ask to reset the password (via [/api/user/resetpwdcreate](api-user-resetpwdcreate)). 
If called Foodsager's server will send an email with a link to reset the password.
**Facebook login**
The client should first make an API call ([/api/user/fbappid](#api-user-fbappid)) to fetch Foodsager's application id, at Facebook.
Once it has the Facebook's appid, the client makes a federate login request to Facebook and gets Facebook's token.
The client then uses the Facebook token to login to Foodsager (via [/api/user/fblogin](api-user-fblogin)). Foodsager will return a valid token of its own. 

## Post login
On success login, the client fetches the user settings (via [/api/user/settings](#api-user-settings)) to get his culture-code. 
Once the client has the user's culture code, he can check: 

* [/api/lang/list](#api-lang-list) - To get current languages client text version. If the client's text version is deferent from the server, the client gets and save the new text via call to: [/api/lang/trans](#api-lang-trans)
* [/api/resource/recipe](#api-resource-recipe) - To get resource for displaying ingredients and Directions.
* [/api/resource/recipecategories](#api-resource-recipecategories) - To get recipe categories in the user's culture-code.

##Start screen
Once all resources where loaded, the user sees the start screen. This screen is divided to four sections:

* Top Left you have "Private Label" recipes.
* Top Right - "My Kingdom". Recipes from the user's cookbook
* Bottom left - Advertising area.
* Bottom Right - Community recipes.

At this point the user can search for recipe, in any of the categories, by filling in text in the edit box. While the user typing, the client calls [/api/search/qa](#api-search-qa) to get some text suggestions for the user.
When the user presses enter the client make a call to [/api/search/all](#api-search-all) to get search results.
> (*new*) "Private Label" Recipes (top left)
A user can only get "Private Label" (here after PL) recipes from special users in Foodsager. 
If the user does not have a "Private Label", recipes assign to him, then this button should act like "My kingdom recipe", moving the user to his own cookbook.
The client will get an array of PL sources, each PL source has a URL to an image that should be displayed on the PL button.
If this array has only one object, then the image should be display on the button. 
If the array holds more then one source of PL,  the client should display their images in a cyclic way, meaning showing the first image for a few seconds, then show the next image and so on.

> Pressing the PL recipe button behaves differently when we have a single PL or multiple PLs:
In case of a single PL, we immediately display the PL recipes.
If we have more then on PL source, the client present a dialog showing the names of all the PLs sources. After the user selects one of them, he gets to the relevant PL recipes.
Private labeled recipes, are read-only and cannot be edit by the user. However, the user may add notes to these recipes.

### "My Kingdom" user's own cookbook

All user's created or imported (apart from private label), recipes, will be displayed here. 
All recipes are displayed as a list view. 
Recipes are displayed two recipes per row. 
Client should use page based strategy, **at client side**, for displaying recipes. 
When client asks for cookbook, he gets a json with **all** user's recipes. 
Since some cookbooks holds thousands of recipes, the client should prepare a list view with the first few recipes (currently set to the first 40 recipes).  
The user can then scroll down the displayed recipes. 
**Before** the user reaches the end of the scrolled view, the client fills up the next page of recipes.
The user can search his own recipe in the designated edit box, above the recipes list view.
> New Feature: When user scroll down the view, his title scrolls out of view, but the search edit box should remain visible at the top of the screen.

Each recipe in the list-view displays the following information:

- Latest picture of the dish.
- Name of the dish
- (*new*) How many people imported the recipe. 
- (*new*) How many people viewed the recipe. 
- (*new*) Average score of the dish (stars) .

In case of imported recipe additional information is displayed:

- Name of the owner of the recipe (on the recipe picture).
- A small circular avatar at the top left corner of the picture (If the owner of the recipe had supplied one). If this value is unavailable, no avatar should be displayed



If the user Wants to share a single or a group of recipes, he does so by pressing the share button at the title. 
All recipes in the view gets a checkbox next to them. The user clicks each recipe he wants to share. At the end of the selection he presses the "Share with" button, and he presented with possible applications he can use to share the recipe.


#### Adding new recipe
For details refer to [Adding new recipe section](#adding_new_recipe)
#### Sharing single/multi recipes
Next to the "Add new recipe" button is a "Share" (![Share button](https://docs.google.com/drawings/d/e/2PACX-1vS9z-W9FdwXG5ytD-TQM41N1dWFNmVQ4uCHnd80qDa9NF8AIBfVUiUG2B5q35fIv5Qxy9LI3eeiZrDd/pub?w=32&h=32) button. When the user presses this button he is redirected to share recipe view. In this view, the user selects the recipe he wants to share and then press "Share" at the bottom. 
> New: Currently the user press, share via Whats app. We would the user to select any social application, that is installed on the mobile, and can send a message.

Client makes an API call to get the message (that contains a link for importing the recipe)
Once the client gets the share link from the server, it will pass the message content to the relevant application to be shared with other people.


> #### (*new*) Context menu
> At the right corner of the search edit box should be a button for selecting categories. When the user presses this button, he can see all available categories with number of recipes in that category (optional). By tapping a category it enter the category text in the search bar and make a search API call (via: [/api/search/mine](#api-search-mine)).

The user can search for a recipe by typing in some text in a designated edit box. While typing, the client makes an API call ([/api/search/q](#api-search-q)) to get possible text the user want to use.
The client calls [api/search/mine](#api-search-mine) to get possible recipes. The client will display a list-view of the search result. 
When done with the filtering, the user can press X to clear the filter.


### Advertising area
This area will occupied by a web-view (a control for displaying web pages). to get the URL to be display in this view, the client makes a call to [/api/adv](#api-adv)
The URL returned from this will set to the advertising view. In case of communication error, a static image should be display in this area.

### Community area
Pressing the community button gets the user to the community recipes. Any recipe that was **not** import by the user and is set as public will be displayed here. 
All public recipe are in read only and the user cannot edit their content. 
The user can import any public recipe to his cookbook.
When client gets a cookbook ([via /api/cookbook/list](#api-cookbook-list)) or explicitly by calling [/api/cookbook/commun](#api-cookbook-commun) it gets the first available recipes in the user's selected language. 
The client should display them in the list view, same as done in the "My Kingdom" recipes. User can scroll the list-view. **Before** getting to the bottom of the list view, the client make a call to [api/cookbook/commun](#api-cookbook-commun) asking to the next 40 recipes. Once getting the answer, client add them to the current list view.
**Note:** Common list view are different from user cookbook in that the server respond with partial list (unlike user own recipe that you get **all** the recipes). There for client should add new recipes as the user scrolls down the list.
Each recipe element holds:

- Name of the owner of the recipe (on the recipe picture).
- A small import icon at the right side of the title.
- (*new*) A small circular avatar at the top left corner of the picture (If the owner of the recipe had supplied one).
- (*new*) How many people imported the recipe. 
- (*new*) How many people viewed the recipe. 
- (*new*) Average score of the dish (stars) .


## <a id="adding_new_recipe"></a>Adding a new recipe
Any user can add new recipe. A user can add a new recipe by pressing the ![plus](https://docs.google.com/drawings/d/e/2PACX-1vQuEl9CTdCxhPmHXRqAswlt5ZeOPukPWsJ1-Je0mxFZJpk3jFjHX6pHCWXmvwmKxYtHHv8sHAHjypRq/pub?w=50&h=50) at the home screen, and the "My Kingdom" screen. 
If the user add from camera or type in a recipe, the client makes a call for creating a recipe, once the client gets successful respond from the server it continues by editing fields in the recipe.
General flow of API to create and edit a new recipe:
![add new recipe flow](https://docs.google.com/drawings/d/e/2PACX-1vR8RxyLJyWcN1kAec9YNfFS743B4mtCMX3_BY-bqQZgTG8bqqjO5nVCErpEN_tenyQPJNcDd4Bl45-X/pub?w=572&h=451)

The user can ask to create a recipe by typing, scanning or (*new*) via website.
If the user wishes to type in the recipe, the client makes a post request to [/api/cookbook/recipe](#api-cookbook-recipe-post) where he can start typing in a recipe.
If the user selects to scan a recipe, he is presented with two options, "take picture" or "from gallery". When satisfy with the image, the client adds the scan to the new recipe.
When done scanning a recipe, the client will display a dialog box that add the following details:
- Recipe title (must)
-  Copyright (must)
- Select recipe category (optional)
- Add recipe picture (optional)

**Note:** User **MUST** set copyright field before he can done edit the new recipe, or he should delete the recipe.
After the initial creation of the recipe (and getting a successful respond), the client makes put request, for each field that the user fills, via [/api/cookbook/recipe](api-cookbook-recipe-put), API call.

>  **New option**: The third option for adding a recipe is via website. When  selected the client opens in-app web view. The in-app browser will enable:
>  1. Show favorites from the device's browser.
>  2. Edit box, for typing in a web URL.
>  3. Back, forward, save buttons.
>  
>  If the user select to save the page he is viewing, the client makes an API call to POST[/api/cookbook/reicpe](#api-cookbook-recipe) passing the URL to import to the user's cookbook.
>  If the respond from the server was good, the user will see the new recipe, in a detailed view at my kingdom cookbook


## Recipe detail view
When the user press a recipe the client makes a call to [/api/cookbook/view](#api-cookbook-view) to mark to the server that a user is watching the recipe.
Follows a detailed review of the recipe.
> **New feature:** The recipe detailed view should be one long scroll-able view. The user can jump to the relevant section of the recipe by pressing a button on the tab-bar which is located at the bottom of the screen (The one with: Recipe details, Ingredients, Instructions). 
**When viewing a recipe, any field that has no value or empty, does not need to be displayed**.
This will make scrolling in the recipe faster.
The user can also enter Edit mode for recipe he added to his cookbook (not "read only" recipes).
#### Title (title)
The title of the recipe .
Second row, bellow the title shows the name of the user who **shared** the recipe recipe (sourceName field)
#### Recipe image (pictures)
Here displayed all the recipe's pictures. Client gets the list ordered by time ascending. The user can scroll between the images by sliding to the sides. 
There should be dots at the bottom of the screen to slide between images. Each recipe can have up to 5 images. 
When viewing a recipe that was import to the user's cookbook, the user will see the user's avatar on top of the image, at the top left corner.

If the user clicks the image, he directed to an image view screen, where he sees the images at full scale and can delete an assigned image of add another one.
Tapping the image again shifts to a "full screen mode, were the user can zoom in on the image.
#### Action bar
Here the user can take actions, or see information about the recipe.
- **delete**:  To delete the recipe.
- **Share**: To share the recipe. On the button, the user can see the number of shares this recipe has (shareCount, scores).
- ** Private Label (new) **: If the user type is Private Label, the he can mark a recipe as Private Label. This way, when he shares the recipe with others, The recipe will be displayed at the "Private Label" section.
- **score**: To see current average score and set the user score. Clicking the score opens a small dialog that shows: The user set score. A diagram with the distribution of the scores (1 point, 2 points and 3 points)
- **Number of views (viewCount)**: Number of views the recipe got. Relevant fields to display this info are: 
####  Recipe details
Follow are the names of the fields in the detail view. Each fields has its correlated field in the JSON format
- **Recipe owner and copyright (copyright)**:  The owner of the recipe. 
- **Recipe Name (title)**:  Title of the recipe 
- **Url source (sourceUrl)**: The URL source of the recipe . Pressing this field opens the web view with the 
-  **Video link (videoUrl)**: If the did not assign (he will see three buttons youtube, vimeo, web). If the user had assign a value, client should display a preview image.
- **Recipe content (categories)**: Categories the recipe belongs to.
- **About the recipe (description)**: Free text, multi line about the recipe.
- **Total preparation time (prepareTime)**: Estimated time it will take to prepare this recipe.
- **Number of servings (servings)**: Estimated number of dishes for these amounts.

#### Ingredients (ingredients)
Each ingredient is displayed in its own line. Each line has four columns:
- **Icon (iconUrl)**: representing the ingredient
- **Text (text)**: Free text describing the ingredient
- **Amount (amount)**: The amount of the ingredient
- **Units (unit)**: The unit of the ingredients.

#### <a id="ingredient-notes"></a>Ingredient Notes (notes.ingredients)
 Ingredient Notes are set by the user, to the ingredients. 
The user can add notes to **ANY recipe (even to read only recipes)**
A note can be in a form of text **or** image.
If the user sets a Text note (field text is set and the note type is zero), It should be displayed as balloon. Each text has its own balloon.
If the user set a picture note (pic,thumbnail are set and note type is 1), the application displays the thumb version of the note's picture.
If the user tap the thumb picture, he is directed to "picture view", showing the image at large, and make actions on the image there.

Note's fields are:
- **id (id)**: Id of the note, use for deleting or updating the note.
- **Note type (noteType)**: 0 (zero)- For text note. 1- For picture note
- **text (text)**: The text of the note. Set only if noteType is zero
- **pic**: Full image size. Set only if noteType is 1
- **thumbnail**: Thumb nail version of the image note. Set only if noteType is 1
#### Directions (directions)
Each direction if made of two parts: direction icon area, direction text
**Direction icon area**: A direction may have up to four icons, representing the action taken in the direction. 
**Direction text**: A multi line text box, holding all direction's text. Text should be wrapped in the text box. An explicit new line is marked with '\n' character (You might need to set a deferment value for iOS.
Direction fields are:
- **Actions (actions)**: An array of actions. Each action is denoted by: (actionsId, iconUrl)
- **Text (text)**: The text of the direction.

#### Scans
A user can scan a recipe to his cookbook. If he does so, this section displays the images.
Displaying a scan behave exactly as picture view.
#### OCR (ocr and ocrDir)
If the user scanned a recipe and it was OCR, these two fields are set as follow:
- **ocr**: Holds the text of the recipe. Lines should be wrapped. Explicit line break is set by the \n character
- **ocrDir**: In a left to right language this field is set to "ltr". In right to left languages, this field is set to "rtl".
The ocrDir is a button at the top of the text. When pressed, the button togles the ocrDir between "rtl" or "ltr"
####Direction Notes (notes.directions)
Same as [ingredient notes](#ingredient-notes), only displayed after the directions.

## Edit recipe
When pressing the edit button, at recipe view, and the recipe is not in "readonly", the user gets to the edit mode view of the recipe.
In this mode, **all recipe's fields are visible**, allowing the user to enter text or set the value, as needed.
When editing edit line or multi line text box or when displaying a text, the client will try to determine the direction of the text according to the two first letters character of the field. The default direction is left-to-right ("ltr"). But if the first (none numeric) character is in Arabic or Hebrew letter, text direction will switch to right-to-left ("rtl").
Follow is a description of the *special* fields in the recipe:
#### Recipe URL (sourceURL)
In edit mode this field should have an edit box and "From browser" button next to it. 
The user can type in a URL to the edit box (normal behavior of an edit box). 
If the user presses the "From browser" button he is switches to the web view where he can look for the page he wants to keep and press save (in the web view). He then redirected back to the edit recipe view.

#### Pictures, Scans and image notes
All recipe scans, pictures and "image notes" should be edit via the [gallery view](#gallery-view). This view shows the image on full screen, allow delete of an image or adding a picture from library of camera
#### Ingredients (ingredients)
Each ingredient is set in a single line. As stated, each ingredient has an icon, text, type of unit and unit values.
The user need only to type in the text of the ingredient. However, when displaying the ingredient, the client needs to take into account all the other fields, meaning that the client need to reserve area for the ingredient icon, the units type and units.
If the none text fields of the ingredient are missing or set to empty, the area should be left blank.
When entering edit mode, the client displays all existing ingredients. At the bottom of the list should be a plus button. Pressing it, will add a new ingredient line (text) that the user can fill in with the keyboard.
#### Directions (directions)
A direction is a multi-line instruction of the completing a step in the recipe. There may be action icons for each direction.
When edit, the user need only to type in the text. However the client should be aware of icons, and so, if present should be displayed with the directions.
When in edit mode, after the last direction, should be a plus button for adding a new direction.

#### Notes edit
**Each** recipe, in the user's own cookbook, should have an option to add notes, to the ingredients or directions. 
Each note can be set as a text or image.
In case of image note, the user sees the thumbnail version of the note. When pressing the thumbnail, the user is directed to the image view.
Notes are considered to be a **quick edit** element. Therefor the user does not need to set the recipe into edit mode, to add a note.
After the ingredient section and the directions section (in case of OCR, then after the OCR), the user will see a bar with two buttons: "Add text note" , "Add image note".
Pressing the text note, will display a multi-line text editor. Once the user done edit, the note will be added to the recipe.
If the user tap the "Add image note", the user will be able to select, "from library or camera" and set the note image. Once the image uploaded to the server, the client gets a link to the thumbnail version.

#### Share recipe
A user can share a recipes from his own kingdom.
If the user created the recipe, he can share it.
If the user imported a recipe, then he can share that recipe only if the recipe is marked as "public".
The general flow of client server interaction is:
![share recipe via social network](https://docs.google.com/drawings/d/e/2PACX-1vSA-wnAn3Q9riKqPMDHyqnAXvqJdxKT2aOjK5vdBdsJRiVf7rzaXmpROf8pWiEEHZguYHz7nKF-Vj7Q/pub?w=535&h=391)
When share recipe button is pressed, the client makes a call to "generate share link" (???? need to set the api end point). Then the client displays all social apps available.
The user selects the application and the client generates a message with the help of the respond from "generate share link" api.
The client will allow the user to add some text of his own to the message.
## Search for a recipe
 A user can search for a recipe when he is at the start screen, "My recipe kingdom" or Community view.
If the user is at "My recipe kingdom" he searches in his cookbook.
If the user is at the community view, then he search the community recipes.
When searching at start screen, then the user searches in both his and community recipes.
When the user types in the text box, the client makes requests to the server to get possible hints, the user can select for quick feel.
Once the user press enter, the client makes a call to the search api.
The api calls are:

**For both community and private cookbook**
/api/search/qa - For getting hints from all recipes
/api/search/all - For searching the entire community

**For private cookbook**
/api/search/q - For getting hints from user's cookbook
/api/search/mine - For searching user's cookbook

**For private cookbook** 
/api/search/qc - For getting hints from public recipes
/api/search/common - For searching public recipes

## Settings
The user gets to the settings screen from the start screen. 
Here the user sets his preferences. The user's settings are:
1. Full name.  (if not register via Facebook).
2. Set/Change/unset User avatar - by getting a picture from gallery or take picture.
2. Change Password (if not register via Facebook).
3. Email.  
4. Favorite kitchen.
5. Language - Client makes an API call to get all supported client side languages.
6. Read only field that state if the user can add "Private Label" recipes.

## <a id="gallery-view"></a>Gallery view (aka image view, picture view or scan view)
A recipe may have pictures, scans and image notes assign to a recipe. 
When in recipe view, or edit, the user should see the gallery view in "embedded" mode. That means small version of the full screen gallery. 
In the gallery, the user sees all images that are relevant to the field (recipe pictures, scan images, ingredient image notes or direction image note).
When tapping this view, the user is shift to full screen gallery view. 
In this mode the user has bars for, deleting current picture add a picture from gallery or take a picture with the camera.
In case of recipe picture, when no picture was assigned to the recipe, a general place holder image should be displayed.
From this mode the user can go back, to the previous view, or tap an image again and see a full screen version the picture, where he can zoom in and out.
(The general functionality of the gallery is as with a mobile device handling display of images)
In any full screen stage, we should have an X button (top right) to close the view and go back to recipe.
## Web view
** This view is new ** and so we will need some graphic design for the view.
This view is an in App web browser that helps the user see recipes that are linked to a webpage, or a way to add recipes from the web.
The general guide line for the view are:
At the top there should be a:
- back button
- Edit box for edit and/or type URLs
- See favorites
  
At the bottom there should be buttons of:
- Go back in history
- Forward in history
- Keep to my FoodSager

We can get to this view from several locations:
- When adding a new recipe from web, 
- When editing a recipe URL link field (sourceUrl field)
- When displaying a recipe from the web (from recipe view).
- When tapping the recipe video
- When editing the recipe video field

If we got to Web view from "Recipe view" the keep button and URL edit box has no meaning and should be disabled.
When editing a recipe Url link the keep button update the recipe sourceUrl field based on current page.
When editing a recipe Video, the keep button updates the videoUrl field, based on current page.
When adding a new recipe "From Web" the save button will create a recipe by calling /api/cookbook/recipe passing the URL that the user selected.

 
## Notification ability
We look for two types of notification to a user, an icon badge number and Application notifications.
### Icon badge
The client should make a call to /api/cookbook/badgecount. The API will return the number of new recipes. If this number is zero, no badge should be displayed. Otherwise the number should be displayed on the application's icon (like with unread messages for email or whats app).

### Application notifications
We wont to be able to send notifications to our users. We will need your help in defining this ability. We know that the client should register with the notification system and then send the server the id it got from the notification server.


## Foodsager’s API

All calls to Foodsager API are RESTful. The general rule is that upload parameters will be in json format, except for when uploading a file. In a file upload case the communication will be as multipart form.
All api calls should be set to host https://www.foodsager.com

GET API calls will have all parameters set in the URL (http standard). For instance to make a call to endpoint /api/user/signup with a parameters of id = 10103 you will send:
http://some.example.com/api/user/signup?id=10103
All POST,PUT and DELETE actions will have a json body (Not converted to form encoding as done by default using jQuery). 

Any call to the server can have two possible outcome success or failure. 
There will be two types of failure: “soft” and “hard”. 
“Soft” failure is when something was wrong with the call but the client could not predict it. The “hard” error is standard HTTP status code.
The server assumes that the client filled the request’s parameter in a correct way, therefore, if one of the parameters is wrongfully set the server’s response will be:  “Bad Request” (400). You will get such error in case you did not fill mandatory field or set invalid data in a parameter.
In case the client tried to make an action, that it unauthorized to take, the server will respond with Unauthorized (401) error ( For instant when the user tries to edit a recipe it does not own).
If the server tries to respond to a client request and some internal error occurs it will respond with Internal Error (500). 

In case of 401 (Unauthorized). The client should attempt to re login via the credentials it has (in case of native login). If the login attempt failed the client should redirect to the login page. 

If the user logged in via FB, the client should show Facebook login screen.

In case of 500 you should display a general error message.
In case of 503 (Service temporarily unavailable), The client should sleep for 2 seconds and try again.

You should use these error codes to decide what the client should do next.
Good or “soft” error response will have the following JSON form:
ok: false in case of “soft” error (true if all is ok). 
msg: In case of “soft” error, a description of what went wrong. The message should be displayed to the user in a message box (with ok button)
payload: Response's information (if any).

NOTE: All calls uses authentication token giving via the /api/user/login??? or api/user/fblogin??? API calls. The token should be passed  in every consecutive request’s header as X-FS-TOKEN. Calls that do not need authorization are marked as “No Token”.


### <a id="resource_recipe"></a>Resource API 
This endpoint will provide the client with various resource information needed by the client for displaying some parts of the application.
The  client should use this endpoint when the application starts. To get the resources you do not need any authentication. 
Client calls these API as start up, after user successful login.

The client should save the resources it got from resource API locally, so it can use them at next login.

#### <a id="api-resource-recipe"></a>Resource for recipe
No Token 
***Method***: GET
***Endpoint***: /api/resource/recipe
**Response**:
``` javascript
{
  "ok": true,
  "payload": {
    "ingredients": [
      {
        "ingType": 18,
        "isNone": true,
        "image": "https://s3.amazonaws.com/drive.foodsager.com/resource/blank.png",
        "units": []
      },
      {
        "ingType": 0,
        "isOther": true,
        "image": "https://s3.amazonaws.com/drive.foodsager.com/resource/other.png",
        "units": [
          {
            "unit": "",
            "amounts": [
              "¼",
              "⅓",
              "½",
              "⅔",
              "¾",
              "1",
              "1¼",
              "1⅓",
              "1½",
              "1⅔",
              "1¾",
              "2",
              "2¼",
              "2⅓",
              "2½",
              "2⅔",
              "2¾",
              "3",
              "3½",
              "4",
              "4½",
              "5",
              "5½",
              "6",
              "6½",
              "7",
              "7½",
              "8",
              "8½",
              "9",
              "9½",
              "10",
              "10½",
              "11",
              "11½",
              "12",
              "12½"
            ]
          }
        ]
      },
      .
      .
      .
    ],
    "directions": [
      {
        "dirId": 0,
        "image": "https://s3.amazonaws.com/drive.foodsager.com/resource/other.png",
        "isOther": true
      },
      {
        "dirId": 1,
        "image": "https://s3.amazonaws.com/drive.foodsager.com/resource/whisk_ic.png"
      },
      .
      .
      .
    ],
    "prepareTime": [
      "0:0:1:0",
      "0:0:2:0",
      .
      .
      .
      
    ]
  }
}
```
The payload contains three fields: ingredients, directions and prepareTime.
Follow is a detail of every field:
***ingredients***: An array of objects. Each object in the array contains:
***ingType***: Number, the ingredient type you should use when inserting a new ingredient.
***iconUrl***: Url to the icon representing the ingType. If not exists use the ingType to set the image according to the resource. If present (not null nor empty) use this URL to display the icon
***units***: Array of possible units available for this ingredient type. Each units item contains:
***unit***: String that symbolize the unit (g,Kg, ml etc.)
***amounts***: Array of floats representing possible values for this unit. When you display the float you should hide the fraction part of number, for instance a float value of 1.0 should be display as 1. 

***directions***: An array of objects, each object in the array contains:
***dirId***: integer representing the icon. This id is inserted in the direction array when editing recipe directions.
***image***: url to an image representing the direction.
***prepareTime***: Array of strings. Each string contains the preparation time in the form of <days>:<hours>:<minutes>:<seconds>. For example, if a recipe takes two and a half hours to prepare then you get the value as: 0:2:30:0

#### <a id="api-resource-recipecategories"></a>Resource for Recipe Categories
Token is optional but preferable so the label will be in the user’s cultureCode. If no token is given server will set the cultureCode base on the client IP. In any error determine the cultureCode of the request, server will use “en-US”.
***Method***: GET
***Endpoint***: /api/resource/recipecategories
**Response**:
``` javascript
{
  "ok": true,
  "payload": {
    "categories": [
      {
        "id": 30,
        "label": "אסיאתי"
      },
      {
        "id": 28,
        "label": "ארוחות בוקר"
      },
      {
        "id": 18,
        "label": "בשר"
      },
      {
        "id": 19,
        "label": "דגים"
      },
      .
      .
      .
    ]
  }
}
```
The payload has a single field call categories.
***categories***: An array of object of:
***id***: An integer to be set for this category.
***lable***: Name of the category in the language set by the user's cultureCode.

### User API
These APIs manage users

####<a id="api-user-signup"></a>Signup
***No Token***
***Method***: POST
***Endpoint***: /api/user/signup
 **Request Parameters**:
```javascript
 {
   "e":"some@email.com",
   "n":"User name (optional)",
   "p":"Password",
   "cc":"en-US",
   "h":""
}
```
***cc***: Culture code of the user such as ‘en-US’,’he-IL’ etc.),
***h***: ”Hash code”, This should be a number (64 bit) that divides by 40641269. The client should
 generate a random number from 1000 to 100000 and multiply it by 40641269. the result should be send to the server. 
**Description**: 
This call signup a new user to the system. The client should make a double edit box to see that the user types in correct password. 
You also need to set a hash code as explain in the parameter section.
**Response**: 
If the hash code could not be determined or if it’s incorrect you will BAD_REQUEST error.
If something was wrong with the email, password you will get soft error with description of the problem in English.
Example of good response:
```javascript
{
   "ok":true,
   "payload":{
      "oh":"421bd8af17ac220426d6511572e96c9f06924c791f9d999f0b51cb38aadc8a4b",
      "name":"sagi test 1", "portraitUrl":"https://s3.amazonaws.com/drive.foodsager.com/production/emptyPortrait.png"
   }
}
```
***oh*** : Auth of the user. From this point on the client should use this token in all its actions.
***portraitUrl***:  Url to the user’s portrait image. If no portrait was set by the user, server will send a default portrait image.

[id]:api-user-login
#### Login
***No Token***
*** Method ***: POST

*** Endpoint ***: /api/user/login

** Parameters **:

*** e ***: “User email”

*** p ***:”password of the user”

*** cc ***:culture code of the user such as ‘en-US’,’he-IL’ etc.)

**Details**: Login the user to the server.	
**Response**: If something was wrong with user/password you get BAD_REQUEST.
If the user password were not found you get a soft error with a message like “Unknown username or password”.

***oh***: Authentication to be used on all calls regarding a user’s cookbook.
***portraitUrl***: A link to the user’s portrait.

If all is well payload will contain JSON object with oh field that will hold the token the client should use for future interaction with the server.

#### <a id="api-user-logout"></a>Logout
***Method***: POST
***Endpoint***: /api/user/logout
**Parameters**: 

**Details**: Will log out current user.
**Response**: 
You get success in any case.

#### <a id="api-user-changepwd"></a>Changing Password
***Method***: PUT
***Endpoint***: /api/user/changepwd
***po***: old password
***pn***: new password

**Description**: This will change the user’s password in the site
**Response**:
If all is OK you receive a new token from the system:
***oh***: new token to be used on all API calls from this point on.

#### <a id="api-user-resetpwdcreate"></a>Create Reset Password 
***No Token***
***Method***: POST
***Endpoint***: /api/user/resetpwdcreate
**Parameters**: 
***e***: user email address.
***cc***: culture code of the user such as ‘en-US’,’he-IL’ etc.

**Details**: When a user forget his/her password he should press a forgot password in the client. He then redirect to a screen where he/she should type in his/her email address. That email address is sent to the server. The server sends an email message with a link to a reset password page.
**Response**: 
If the email is empty you get a BAD_REQUEST (400).
If the email is unknown you get failure response with a msg.
If the email address is valid you will get a success response with no payload. 

####<a id="api-user-fbappid"></a>Get FoodSager Facebook’s application id
 ***No Token***
***Method***: GET
***Endpoint***: /api/user/fbappid
** Description**:
To give credentials to FoodSager via Facebook (Hereafter FB), the clients needs FoodSager’s AppId on FB. You call this endpoint.
**Response**:
```javascript
{
   "ok":true,
   "payload":{
      "appid":"1483851515263121"
   }
}
```


#### <a id="api-user-fblogin"></a> Login/Register Via Facebook
***No Token***
***Method***: POST
***Endpoint***: /api/user/fblogin
**Parameters**: 
***fbid***: User’s facebook id.
***token***: The token the client got from FB after approving FoodSager application.

**Description**: A user can sign up or login via FB. the client should have the SDK to login and auticating FoodSager by getting FoodSager’s Facebook’s id before login the user via FB. Once the client done with FB login process he gets the user’s Facebook ID and a token.
The client then pass the FB id and the token to FoodSager server via this endpoint.
If the user exists in the system (sync via email) then you will be logged in to FoodSager. If the user does not exists, it will be created and the client will receive a FoodSager’s token.
**Response**:
***oh***- FoodSager auth to be used on future response.
***name***: The name of the user (as registered on FB).
***portraitUrl***: The url from FB for the user’s portrait.

#### <a id="api-user-ping"></a>Ping
***Method***: GET
***Endpoint***: /api/user/ping

**Description**: This method checks if FoodSager's token is valid.
**Response**:
Success if the token is valid.
If The token is invalid you get the usual UNAUTHORIZED

#### <a id="api-user-settings"></a>Settings
***Method***: GET
***Endpoint***: /api/user/settings

**Description**: Get the user’s settings.
**Response**:
On success the payload contains:
``` javascript
{
  "ok": true,
  "payload": {
    "email": "sagi.test1@mailinator.com",
    "name": "sagi test 1",
    "lang": "he-IL",
    "emailVerified": false,
    "isPrivateLabel" false,
    "portraitUrl": "<url>",
    "favoriteKitchen", "favorite kitchen"
  }
}
```
***email*** Email address of the user
***isEmailVerified***: true if the user had verified this email address. If the user had verified the email address he cannot change it.
***name***: name of the user
***lang***: culture code of the user’s interface.
***portraitUrl***: A url to the user’s portrait.
***favoriteKitchen***: String holding the type of kitchen the user likes.

#### <a id="api-user-settings-put"></a>Settings
***Method***: PUT
***Endpoint***: /api/user/settings

**Parameters**: 
***email*** - (optional) A new email for the user. The server checks if the user had not verified the existing email address. If the user had not verified the email address and this email value is valid, then the new email address is set to the user.
***name***: (optional) A new name of the user.	
***lang***: (optional) A new language code.
***favoriteKitchen*** - (Optional) String holding the type of kitchen the user likes.
**Description**: Set one or more field of the user’s settings.
**Response**:
On success you get success response with no payload.

#### <a id="api-user-settings-post"></a>Settings
**Method**: POST
**Endpoint**: /api/user/settings
**Consume**: multipart/form-data

**Parameters**: 
*file* - a file to upload to the server containing the portrait image of the user.
**Description**: Set the user’s portrait image
**Response**:

#### <a id="api-user-balance"></a>Get user coin (deprecated)
**Method**: GET
**Endpoint**: /api/user/balance

**Description**: Gets the user coin balance
**Response**:
*payload*:
 *coin* - The amount of coins in the user’s balance.

 
### Search API
All search and hints results are in the same form as with /api/cookbook/list
**NOTE**: Result may come from both “mine” and “common” areas.

#### <a id="api-search-q"></a>Hints from user own cookbook
**Method**: GET
/api/search/q?t=<search term>
**Description**: search user own cookbook.
**Response**:
``` javascript
{
  "ok": true,
  "payload": {
    "hits": [
      "SHEMO  הקונדיטוריה - מתכון החודש"
    ]
  }
}
```
payload contains hits field with an array of possible text

#### <a id="api-search-qc"></a>Hints from community  cookbook
**Method**: GET
/api/search/qc?t=<search term>
**Description**: Get hints from user own cookbook.
**Response**:
``` javascript
{
  "ok": true,
  "payload": {
    "hits": [
      "SHEMO  הקונדיטוריה - מתכון החודש"
    ]
  }
}
```
payload contains hits field with an array of possible text

#### <a id="api-search-qa"></a>Hints from all cook books
**Method**: GET
/api/search/qa?t=<search term>
**Description**: Get hints from community cook books.
**Response**:
``` javascript
{
  "ok": true,
  "payload": {
    "hits": [
      "SHEMO  הקונדיטוריה - מתכון החודש"
    ]
  }
}
```
payload contains hits field with an array of possible text

Search all cookbook books
Method: GET
/api/search/all?t=<search term>
Description: search all cook books.

Search user own cookbook
Method: GET
/api/search/mine?t=<search term>
Description: search user own cookbook.

Search community cookbook
Method: GET
/api/search/common?t=<search term>
Description: search community cookbook.

Search all cook books
Method: GET
/api/search/all?t=<search term>
Description: search all cook books.




Language API
This api let you query Foodsager’s supported languages and get the translation page of a specific language.
NOTE: Language translation that is implemented by the server is one option of doing things. If your client environment need to be done. 

Get a list of all supported languages
No Token
Method: GET
Endpoint: /api/lang/list
Description: Gets all languages supported by the application. Each language has a cultureCode,name and version. The client should periodically (TBD) query the server to see if there is a difference between the local version and the servers. If so, the client should get the new version using the get translation api call.
Response:
On success the payload contains:
languages - An array with items objects. Each item has the following info:
	lang - culture code
	name - Name of the language.
	version - Version of the translation as a Number.

Get translation text
No Token
Method: GET
Endpoint: /api/lang/trans
Description: Gets the translation of the UI in a supported language (passed by a paramater).
Parameters: 
lang - The cultureCode of the translation to get.
Response:
On success the payload contains:
text - An array that the English version is the key and the value of the item if the translation. For instance, if we have an English label as “hello” then the spanish translation will be “olla”. You get to this text by the following javascript call:
	text[“hello”]


 
Cookbook API
After the login the user can start making request for getting recipes from the server. All request should have an oh parameter that holds the token given at login time.

Get start page information
Method: GET
Endpoint: /api/cookbook/list  or  /api/cookbook/
Parameters:
Details: This call will get the information needed to fill the start screen of the application. 
Response:
If all is OK the payload will contain the information needed for the user’s cookbook, and the community cookbook. 
The payload will contain two types of cookbook:
mine: The users cookbook as an array of JsonObject with the following fields:
id : id of the recipe
version: an integer with the current version of the recipe.
copyright: String with the name of the person who created the recipe. If the user did not set a full name then we will use the part of the user before the @ mark of the email address as the name of the owner of the recipe.
sourceUrl: String showing the web origin of the recipe. This field can be empty.
title: Name of the recipe.
categories: Array of integer IDs of categories this recipe belongs to. To get the category text name look at API /api/resource/recipecategories.
description: Free text about the recipe.
ingredients: An Array that each element is a JsonObject with the following fields:
	amount: 1
	unit: 
       empty= No units are needed.
       g = gram
	       kg=Kilogram
	       l= liter
	       ml=mililiter
       L= Liter

    	ingType:type of ingredients
	      0= None
	      1= tea spoon
	      2= spoon
	      3=cup
	      4=weight
	      5= Fluid Volume
	      6= egg
            text:  Free text about the ingredient
Directions: An array that each element contains:
	actions: an array of object, each action object has the form:
actionId- a number representing the action 
iconUrl- A url of the icon representing the action.
text: A text on this step in the directions.
	action: DEPRICATED, ignore this field empty field of empty array. otherwise this should be an array of numbers. Each number will be represent by an icon. To get the relevant icon attached to the number make a call to resource api.
	

prepareTime: Time for preparing the recipe in the form of <day>:<hours>:<minutes>:<seconds>.
scans: An array of Json Object with:
	id: The id of the image.
	url: A url for getting the image.
	thumb: A url for getting the image’s thumbnail.
pictures:An array of Json Object with:
	id: The id of the image.
	url: A url for getting the image.
	thumb: A url for getting the image’s thumbnail.
yummy: Number of yummy the recipe has.
isYummy: Did the user yummy this recipe.
isPublic: True if this recipe is marked as public by the owner.
isFavorite: True if the user favore this recipe over the others.
readonly: True if the user cannot edit this recipe. This field is set to True if the user is not the owner of the recipe.
isNew: True if the user had not seen this recipe. (Always false for recipes that the user created).
isForeign: True if this recipe does not belong to this user.
sourceName: (Optional)If isForigen is true this field may contain the name of the user who gave the user the recipe (ether my user import or source user pressed share).


community: Has all mine recipe fields and a new field called cookbookUserName that holds the name of the user the cookbook belongs to.

newRecipeCount: Number of new recipe shared with the user and had not viewed.
mineRecipeCount: Total recipe in user’s own cookbook. This will be useful if client use server side pagination.

notes: A object containing sub fields:
	ingredients: An array of notes related to ingredients. Each note contains a note info. 
	direction: An array of notes related to directions. Each note contains a note info. 
	
Get user’s cookbook
Method: GET
Endpoint: /api/cookbook/mine
Details:
getting the user’s own recipes only. 
Response:
See /api/cookbook/list api call. This will be the same without the community part
Get community cookbook
Method: GET
Endpoint: /api/cookbook/community

Details:
getting the community recipes only. 
Response:
See /api/cookbook/list api call. This will be the same without the mine part

Get Unviewed new recipe
Method: GET
Endpoint: /api/cookbook/newcnt
Parameters:
Details: How many recipe shared with the user and he/she did not view them yet.
Response:

newRecipeCount: Number of recipe the user had not viewed yet.

Create new recipe
When the user hits the “Add recipe” button the client should immediately make this API call and get the recipe new id. 
From this point on the client should send PUT calls to update the new recipe.
Method: POST
Endpoint: /api/cookbook/recipe
Parameters:

copyright: optional, Source of recipe. If this field is empty system will return 
Response:
id: new Recipe id
copyright: The copyright generated by the system.
title: The default title of the recipe. Client must set it with the one set by the user.
rest of recipe fields are set to their default empty values.

Description: 

Update recipe 
Method: PUT
Endpoint: /api/cookbook/recipe
Parameters:
id:<recipe id>
copyright: optional, new string of the recipe creator.
sourceUrl: optional , the web resource this recipe came from.
title: optional, title of the recipe.
description: optional, description of the recipe.
prepareTime: optional, set the new value in form of <days>:<hours>:<minutes>:<seconds>.
ingredients: optional JSON array of all the ingredients of the recipe.
directions: optional JSON array of all the directions of the recipe.
servings: optional, set new number of servings.
isPublic: optional, True if the recipe should be public, false if it should be private.
isFavorite : optional, True if the user likes this recipe.
ocr: optional, Text of the OCR field of the recipe.
Details: This call update recipe information. except for the id, all fields are optional. Only the fields that are part of the request will be updated on the server. Client should call this whenever a field is changed on the client. When a field is changed the client will set the relevant field to the new value and PUT it in the server. Client should also change its local copy of the recipe info.

Response:
ok- Will be True if records were updated. 
If the user try to update a recipe he does not own you will get http error code 400 (bad request)
payload- Contains the updated recipe information (see create new recipe for details of the fields).


 
Delete recipe
Method: DELETE
Endpoint: /api/cookbook/recipe
Parameters:
id:<recipe id>

Description:
This call will delete a recipe from the user’s cookbook. 
When the user delete a recipe it may have impact on the community recipe list, for instance: If the user delete a recipe that he imported from the community, then the delete recipe will switch to the community recipes from the user’s perspective.
For this reason, after recipe deletion you get the new image of the users cookbook.

Response:
ok- True if the recipe was delete.
payload- 
mine:The user’s private cookbook (Same as “mine” object when you list the user’s cookbook)

Upload picture:
Method: POST , multipart/form-data
Endpoint: /api/cookbook/picture
Parameter: 
id: recipe id
file: Image file to upload to server. file image should be up to 8 mega


Endpoint
/api/cookbook/picture
Method
POST
Content-Type
multipart/form-data
Parameter 
id: Id of recipe that the new picture belongs to.
file: Image file to upload to server.
clientInfo: A string containing information that will be send back to client if all goes well.
Response
{pictures:the new picture array in a JSON format.,
clientInfo: The information set by the client at the call}




Delete picture:
Method: DELETE
Endpoint: /api/cookbook/picture
Parameter:
id: recipe id
pic: picture id
Response:
pictures: the new picture array in a JSON format.
Upload scan:


Endpoint
/api/cookbook/scan
Method
POST
Content-Type
multipart/form-data
Parameter 
id: Id of recipe that the new picture belongs to.
file: Image file to upload to server.
clientInfo: A string containing information that will be send back to client if all goes well.
Response
{scans:the new scan array in a JSON format.,
clientInfo: The information set by the client at the call}




Delete scan:
Method: DELETE
Endpoint: /api/cookbook/scan
Parameter:
id: recipe id
scan: scan id
Response:
scans:the new scan array in a JSON format.
Set Yummy
Method: PUT
Endpoint: /api/cookbook/yummy
Parameters: 
id = recipe id
isYummy = true if client wants to set yummy, false if unyummy.


 
Add note

Method: POST
Endpoint:/api/cookbook/note
Content-Type: application/json

paramteres:
recipeId - The id of the recipe to add this note.
noteType- 
	0 - A note related to ingredients
	1 - A note related to directions (instructions)
text - The text of the note.

Description:
This method adds a text note to a recipe, in the user’s kingdom.

If all is OK, the call returns:

id - The id of the new note.
“text” - The note’s text


Add note

Method: POST
Endpoint:/api/cookbook/note
Content-Type: multipart/form-data

paramteres:
recipeId - The id of the recipe to add this note.
noteType- 
	0 - A note related to ingredients
	1 - A note related to directions (instructions)
file - The image file to upload to the server.


Description:
This method adds an image note to a recipe, in the user’s kingdom.

If all is OK, the call returns:
id - The ID of the new note.
pic - A URL to the note image.
thumbnail - A URL to the note thumbnail


Delete note
Method: POST
Endpoint:/api/cookbook/note
Parameters: 
id - ID of the note to delete.

Description
This method delete a note from a recipe. 





 
View recipe
Method: PUT
Endpoint: /api/cookbook/viewed

Parameter: 
id: recipe id
Description: When a user looks at a recipe the client will notify the server about this. 
Response:
If all is well you get an ok set to true, and number of recipe the user had not seen. 
If something goes wrong you get a ok false with description of what went wrong.
Import recipe
Method: POST
Endpoint: /api/cookbook/import
Parameter: 
id: id of recipe the user wishes to import from the community recipes.


Description: User can import common community recipes using this API.
Respons:
If recipe was imported to the user’s cookbook you get a payload with “mine” field containing the updated version of user’s own cookbook.

Import recipe
Method: POST
Endpoint: /api/cookbook/import
Parameter: 
id: recipe id
Description: 
This API import a recipe from the community cookbook to the user’s cookbook.
response:
If the recipe was successfully imported, the payload contains a “mine” object containing the user’s cookbook.
A failure may occur if the user try to import his own recipe or if the imported recipe is not marked as public.

Share recipe
Method: POST
Endpoint: /api/cookbook/share
Parameter: 
id:recipe id
e: Target person's email.
Description: When a user shares a recipe the server looks for the target user via the e field. 
Respons:
If system was able share or send invitation 

Share a recipe on Facebook 
Method: POST
Endpoint: /api/cookbook/fbshare
Parameter: 
id:recipe id
Description: Posting on the user’s wall on facebook
Response:
If the user was not logged in via FB, you get a message telling you that the user should do so.


 
Share a recipe with WhatsApp
This API will create a message that contains a link to a webpage. The webpage will open Foodsager and run the url command.

Endpoint
/api/cookbook/washare
Method
POST
Content-Type
application/json
Parameter version 1
{id:<recipe Id>} For sharing a single recipe
Parameter version 2
{list:[<recipe id 1>,<recipe id 2>]} For sharing several recipe in one call, use list parameter name and pass an array of recipe ids
Response
{msg:<default message to send the target whatsapp user>,link:<url to command>}

Remarks:
The msg is the default message that set in the WhatsApp edit box. After the user set the final text, Foodsager need to verify that the link url is set in the text. If it is not, client will append the link to the message.


 
## Language API
Follow are the REST for multi-language support

#### <a id="api-lang-list"></a> Get supported languages
- *Method*: GET
- *Endpoint*: /api/lang/list

**Description**: 
Getting list of supported languages. 

**Response**: 
```javascript
{
   "ok":true,
   "payload":{
      "languages":[
         {
            "lang":"zh-CHS",
            "name":"中文",
            "version":1
         },
         {
            "lang":"en-US",
            "name":"English",
            "version":27
         },
         {
            "lang":"ms-MY",
            "name":"Bahasa Melayu",
            "version":2
         },
         {
            "lang":"he-IL",
            "name":"עברית",
            "version":45
         },
         {
            "lang":"ja-JP",
            "name":"日本語",
            "version":1
         }
      ]
   }
}
```
Languages: an array of language info objects
Each language object contains:
*lang*: Language code.
*name*: Text representation of the language
*version*: The number of the active version of the language. If the client version is different from the server, it should get the updated text table via call to [/api/lang/trans](#api-lang-trans)


####<a id="api-lang-trans"></a> Get language table
- *Method*: GET
- *Endpoint*: /api/lang/trans
- *Parameters*: 
```javascipt
{lang:<language-code>}
```
You can see possible language calls by making a REST to [/api/lang/list](#api-lang-list) and use the lang field

**Description**:
If the client needs to get the latest text table of an application he make this call. If all is OK, the payload will contain the information the client needs. As of now we had not defined the language format. Ok field of the response will be false if the language is not supported.
**Response** (partial)
```javascript
{  
   "ok":true,
   "payload":{  
      "text":{  
         "auth":{  
            "sign_in":"Log In",
            "sign_up":"Sign Up",
            "type_your_email":"Type your e-mail",
            "choose_password":"Enter password",
            "confirm_password":"Confirm password",
            "connect_with":"Connect with",
```
The payload contains a single text object that holds all the static text of the application in the English language.
> If you need to change this translation approach please state and we can make the adjustments
 
### Command API
A user can start Foodsager’s client by pressing a link that has protocol set to foodsager (ex: foodsager://www.foodsager.com)

If the client is stated this way, it needs to authenticate the user, and then immediately call the relevant command api.
 
Executing command via link
Whenever a client starts doe to pressing a url with foodsager protocol, it will make this API call to the server, passing the url.

Endpoint
 /api/cmd
Method
POST
Content-Type
application/json
Parameter
{u:<url>}
Response
Success empty response or in case or soft error.

Remarks:
u- contaning the URL the application received by the protocol handler. For example. If the user hit  the following link foodsager://www.foodsager.com/api/cmd?asdqweasdasdqwasd . The client will pass: foodsager://www.foodsager.com/api/cmd?asdqweasdasdqwasd in the u parameter
<!--se_discussion_list:{"ON7u8WBcffqwoLdp6D3ufy6v":{"selectionStart":5590,"type":"conflict","selectionEnd":5608,"discussionIndex":"ON7u8WBcffqwoLdp6D3ufy6v"},"Ti3tre6ddicfNBSR5ZMSIP5m":{"selectionStart":5699,"type":"conflict","selectionEnd":5719,"discussionIndex":"Ti3tre6ddicfNBSR5ZMSIP5m"},"MpO8FXzEh8uaDLUHicQPSTwr":{"selectionStart":6290,"type":"conflict","selectionEnd":6306,"discussionIndex":"MpO8FXzEh8uaDLUHicQPSTwr"},"G8wkZKPaBKjSx1Av2Zp2C1Gl":{"selectionStart":6469,"type":"conflict","selectionEnd":6490,"discussionIndex":"G8wkZKPaBKjSx1Av2Zp2C1Gl"},"g1HtOmaeBQkxkgtb0uqP1Ypa":{"selectionStart":6951,"type":"conflict","selectionEnd":7000,"discussionIndex":"g1HtOmaeBQkxkgtb0uqP1Ypa"},"yPOB3s3eepBxkNkCI0pSAK46":{"selectionStart":7152,"type":"conflict","selectionEnd":7192,"discussionIndex":"yPOB3s3eepBxkNkCI0pSAK46"},"OaJUUNfiaSy8wihqneoqKOb6":{"selectionStart":7277,"type":"conflict","selectionEnd":7347,"discussionIndex":"OaJUUNfiaSy8wihqneoqKOb6"},"XBY3fsPbLU4G1O4F8VuvXHna":{"selectionStart":7965,"type":"conflict","selectionEnd":8002,"discussionIndex":"XBY3fsPbLU4G1O4F8VuvXHna"},"15HtgVmfp1cmD1wPBVYKclK3":{"selectionStart":8248,"type":"conflict","selectionEnd":8257,"discussionIndex":"15HtgVmfp1cmD1wPBVYKclK3"},"3eoc1TTaj5MCRDgEVxlB282i":{"selectionStart":8317,"type":"conflict","selectionEnd":37781,"discussionIndex":"3eoc1TTaj5MCRDgEVxlB282i"}}-->
<!--se_discussion_list:{"ON7u8WBcffqwoLdp6D3ufy6v":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"ON7u8WBcffqwoLdp6D3ufy6v"},"Ti3tre6ddicfNBSR5ZMSIP5m":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"Ti3tre6ddicfNBSR5ZMSIP5m"},"MpO8FXzEh8uaDLUHicQPSTwr":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"MpO8FXzEh8uaDLUHicQPSTwr"},"G8wkZKPaBKjSx1Av2Zp2C1Gl":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"G8wkZKPaBKjSx1Av2Zp2C1Gl"},"g1HtOmaeBQkxkgtb0uqP1Ypa":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"g1HtOmaeBQkxkgtb0uqP1Ypa"},"yPOB3s3eepBxkNkCI0pSAK46":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"yPOB3s3eepBxkNkCI0pSAK46"},"OaJUUNfiaSy8wihqneoqKOb6":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"OaJUUNfiaSy8wihqneoqKOb6"},"XBY3fsPbLU4G1O4F8VuvXHna":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"XBY3fsPbLU4G1O4F8VuvXHna"},"15HtgVmfp1cmD1wPBVYKclK3":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"15HtgVmfp1cmD1wPBVYKclK3"},"3eoc1TTaj5MCRDgEVxlB282i":{"selectionStart":62377,"type":"conflict","selectionEnd":0,"discussionIndex":"3eoc1TTaj5MCRDgEVxlB282i"}}-->

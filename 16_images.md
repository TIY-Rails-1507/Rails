# Images

In this section we are going to 'jazz-up' the details page of the hotel.

## Task 1

Google for a few hotel images and save them to `app/assets/images/`.

Choose one to be used as the default image.

Please don't waste too much time on picking the perfect images...

## Task 2

Add a new migration that will add a new column to the hotels table. This should be called 'image_name' and should be of type string.

## Task 3

Update the 'edit' form so that we can set an image name. 

Use the text_field form helper method.

When you enter the full file path, simply enter the name of the file.

This assumes you have saved the images to `app/assets/images/`.

## Task 4

Make the hotel details view show the image.

Ref:
* http://apidock.com/rails/ActionView/Helpers/AssetTagHelper/image_tag


## Task 5

Use a custom helper to show the image if it is present or a default image if it is not present.



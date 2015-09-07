# Authorization Challenge!!!

In this challenge we prevent unauthorized access to parts of the application. Specifically we are going to restrict a user so that they can see their own profile but none of the other users profiles.

To achieve this you need to use the existing functionality you have already built and you need to make use of the `before_action` mechanism in the controller.

## Step 1

Google "rails before_action" so that you understand what it does.

## Step 2

When a user attempts to access a profile page
* redirect them to the login if they are not logged in
* let them see the profile page if they are logged in

## Step 3

Redirect a user to their own profile page if they try to access a profile page that is not there own

## Step 4

Make sure that only logged in users can add 'reviews'.
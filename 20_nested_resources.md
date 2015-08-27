# Nested Resources

In Questionable we would like to see all the answers for a Question by using the following URL: 

http://localhost:3000/questions/1/answers

As you can see, this has a 'nested' structure.

Trying this URL at the moment will result in this error:

```
No route matches [GET] "/questions/1/answers"
```

While we can define our own custom routes, it is often easier to use the 'resources' method in rails.

At the moment out routes have this:

```
# app/config/routes.rb

Rails.application.routes.draw do
  resources :questions

```
To created nested routes which support the URL above, we use:

```
# app/config/routes.rb

Rails.application.routes.draw do
  resources :questions do
    resources :answers
  end
```

Using `rake routes` we can now see that we have the desired routes available. 

```
$ rake routes
              Prefix Verb   URI Pattern                                        Controller#Action
    question_answers GET    /questions/:question_id/answers(.:format)          answers#index
                     POST   /questions/:question_id/answers(.:format)          answers#create
 new_question_answer GET    /questions/:question_id/answers/new(.:format)      answers#new
edit_question_answer GET    /questions/:question_id/answers/:id/edit(.:format) answers#edit
     question_answer GET    /questions/:question_id/answers/:id(.:format)      answers#show
                     PATCH  /questions/:question_id/answers/:id(.:format)      answers#update
                     PUT    /questions/:question_id/answers/:id(.:format)      answers#update
                     DELETE /questions/:question_id/answers/:id(.:format)      answers#destroy
           questions GET    /questions(.:format)                               questions#index
                     POST   /questions(.:format)                               questions#create
        new_question GET    /questions/new(.:format)                           questions#new
       edit_question GET    /questions/:id/edit(.:format)                      questions#edit
            question GET    /questions/:id(.:format)                           questions#show
                     PATCH  /questions/:id(.:format)                           questions#update
                     PUT    /questions/:id(.:format)                           questions#update
                     DELETE /questions/:id(.:format)                           questions#destroy


```

This provides us with helper methods which can be used as follows:

```
question_answers_path(question_id) # show all questions
question_answer_path(equestion_id, answer_id) # within the scope of a specific question, show an answer
```


After refreshing the browser end up with: 

```
uninitialized constant AnswersController
```
This means we are on the typical track to fleshing out the required controller, actions and views to make this page work.


## Exercise 

Show all the reviews for a hotel at a URL like this: 

http://localhost:3000/hotels/1/reviews

Make sure that you fetch the hotel and then find the review from that hotels reviews.

e.g. @hotel.reviews


Add a link to the show page to 'see all reviews'

## Extra

To limit the routes being generated see here: 

http://guides.rubyonrails.org/routing.html#restricting-the-routes-created

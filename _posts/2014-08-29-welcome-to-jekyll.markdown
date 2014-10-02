---
layout: post
title:  "Koala Facebook and Rails, oh my"
date:   2014-10-2 14:34:25
categories: jekyll update
tags: featured
image: /assets/images/woods.jpg
---

<div>First things first... we are going to need to install 2 gems to make our
facebook connection work. We are going to use dotenv-rails and koala.</div>
To the Gemfile

    gem 'dotenv-rails'
    gem 'koala', '~> 1.10.0rc'

Step 2... we need to create a .env file that we will put our facebook app_id and
facebook_secret key into. This information is now in the environment and
dotenv-rails will be able to fetch it....very good news

    FACEBOOK_APP_ID='Your Facebook App id'                     
    FACEBOOK_SECRET='Your Facebook secret key'    

Wait a second...where do we get these keys? Create an app on the Facebook
Developers page and Facebook will be kind enough to give you your very own
app_id and a secret key.

 REMEMBER PUT THE .env FILE INTO THE .gitignore IF YOU
DONT THE SECRET KEY IS NOT GOING TO BE A SECRET!

Next we are going to show the user a link to sign into facebook:

    link_to("Login", Koala::Facebook::OAuth.new(ENV.fetch('FACEBOOK_APP_ID'), ENV.fetch('FACEBOOK_SECRET'), sessions_url).url_for_oauth_code) %>

Koala::Facebook:OAuth.new takes three arguments, the app id and the app_secret which we fetch using
dotenv-rails and where we want Facebook's response to be targeted. I chose to
send facebook's response to the sessions_url which will trigger Sessions.new.

Recap: we used the Koala gem to send a login request to facebook and the
response is going to the Sessions controller... what are we going to do now?

Sessions Controller:

    access_token = set_access_token                         
    user = FacebookUser.new(access_token).find_or_create    
    session[:facebook_id] = user.facebook_id                
    session[:name] = user.name                              
    redirect_to :dashboard                                  



    def set_access_token 
      if params[:code]  
        Koala::Facebook::OAuth.new(ENV.fetch('FACEBOOK_APP_ID'),ENV.fetch('FACEBOOK_SECRET'), sessions_url).get_access_token(params[:code])
      end
    end                                                                         

FacebookUser.new is a service method that we are going to create and use to either find a
user who is already in our database or persist a new user. 

    class FacebookUser                                                    
                                                                          
      def initialize(access_token)                                        
        graph = Koala::Facebook::API.new(access_token)                    
        user = graph.get_object("me")                                     
        @facebook_id = user["id"]                                         
        @name = user["first_name"] + " " + user["last_name"]              
      end                                                                 
                                                                          
      def find_or_create                                                  
        User.where(facebook_id: @facebook_id).first_or_create(user_params)
      end                                                                 
                                                                          
      def user_params                                                     
        {                                                                 
          name: @name,                                                    
          facebook_id: @facebook_id                                       
        }                                                                 
      end                                                                 
    end                                                                   

In my very simple app outline I only wanted users to have a facebook_id and a
name. My schema looks like this

    create_table "users", force: true do |t|                                      
      5     t.string   "facebook_id"                                                    
      6     t.datetime "created_at"                                                     
      7     t.datetime "updated_at"                                                     
      8     t.string   "name",        null: false                                       
      9   end                                      


Great we have sent a request to Facebook, got a response directed to the
sessions new controller, either found the user or created him in our database
now we can look back at the Sessions controller to see where to go next


Sessions Controller:

    access_token = set_access_token                         
    user = FacebookUser.new(access_token).find_or_create    
    session[:facebook_id] = user.facebook_id                
    session[:name] = user.name                              
    redirect_to :dashboard                                  




We want to store the user's facebook_id as well as their name in the session so
we can call them all over our wonderful app         

---Chris Thorsen


[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help

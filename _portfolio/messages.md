---
layout: post
title: Kingdom Messages
feature-img: "img/sample_feature_img.png"

thumbnail-path: "img/messages_thumbnail.png"
short-description: Game of Thrones message board

---
[Kingdom Messages](https://information-pam-willenz.herokuapp.com/) is a message board for the Game of Thrones Kingdom, featuring posts, comments, bootstrap-card format, TDD and user authentication. 

<img src="/img/messages_home_page.png">
<br>

<h3> Add Message Modal </h3>

<img src="/img/messages_add_message_modal.png">
<br>

<h3> Message Show Page </h3>

<img src="/img/messages_show_page.png">
<br>

<h3> Requirements </h3>
1. Users can register and then must be authenticated before logging in.
  + Users should have an email (used for logging in), and a first and last name at a minimum. 
 + Users can create posts (title, body, author (user_id)
  + Users can comment on other users’ posts (Comments have post_id, body, author (user_id))
2. There is a posts index page that lists all posts (for entire site). This is the ‘homepage’.
  + This should display a list of posts with the title and the author’s name
  + Order posts with most recent posts at the top of the page
3. There is a post show page, that shows a single post. Underneath the post are all the
comments, ordered by date created (oldest at the top)
4. When commenting on a post, use a modal to open up. 
5. Use RSpec unit tests and at least one feature test, using rspec/capybara - used 31 unit tests

<img src="/img/messages_signup.png">
<br>

<h3> Implementation </h3>

Modified Devise to include both first and last name. Tested models, controllers and various featuresUsed three feature tests to demonstrate capybara:

`Signin Feature Spec`
{% highlight ruby %}

require 'rails_helper'

feature "user registers for site" do

  scenario "user registers" do
    visit root_path
    click_link "Sign Up"
    fill_in_registration_fields
    expect(page).to have_content("Welcome! You have signed up successfully.")
end

def fill_in_registration_fields
    fill_in "user[first_name]", with: "Jamie"
    fill_in "user[last_name]", with: "Lannister"
    fill_in "user[email]", with: "jamie@kingslanding.com"
    fill_in "user[password]", with: "dragon"
    fill_in "user[password_confirmation]", with: "dragon"
    click_button "Sign up"
  end
end

{% endhighlight %} 

`Comments Controller Spec`

{% highlight ruby %}

require 'rails_helper'

RSpec.describe CommentsController, type: :controller do 
  
  describe "comments#create action" do 
    it "should allow users to create comments on posts" do 
      p = FactoryGirl.create(:post)
      user = FactoryGirl.create(:user)
      sign_in user

      post :create, post_id: p.id, comment: { message: 'awesome post'}

      expect(response).to redirect_to root_path
      expect(p.comments.length).to eq 1
      expect(p.comments.first.message).to eq "awesome post"
    end

    it "should require a user to be logged in to comment on a post" do 
      p = FactoryGirl.create(:post)
      post :create, post_id: p.id, comment: { message: 'awesome post'}
      expect(response).to redirect_to new_user_session_path
    end

    it "should return http status code if the post isn't found" do 
      user = FactoryGirl.create(:user)
      sign_in user
      post :create, post_id: 'SPOCK', comment: { message: 'awesome post'}
      expect(response).to have_http_status :not_found
    end
  end
end

{% endhighlight %}

`User Model Spec`

{% highlight ruby %}

require 'rails_helper'

RSpec.describe User, type: :model do 
  describe 'invalidations' do
    it "has a valid factory" do 
      FactoryGirl.create(:user).should be_valid
    end

    it "is invalid without a firstname" do 
      FactoryGirl.build(:user, first_name: nil).should_not be_valid
    end
    
    it "is invalid without a lastname"  do  
      FactoryGirl.build(:user, last_name: nil).should_not be_valid 
    end

    it "is invalid without an email" do 
      FactoryGirl.build(:user, email: nil).should_not be_valid 
    end

    it "is invalid without a password" do 
      FactoryGirl.build(:user, password: nil).should_not be_valid
    end
  end

  describe "associations" do 
    it { should have_many(:comments) }
    it { should have_many(:posts) }
  end
end

{% endhighlight %}

`User Model`

{% highlight ruby %}

class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  has_many :posts
  has_many :comments

  validates_presence_of :first_name, :last_name, :email, :password
  
  validates :first_name, presence: true, length: {maximum: 30}
  validates :last_name, presence: true, length: {maximum: 30}       
end

{% endhighlight %}

<h3> Conclusion </h3>
This app was part of a coding challenge, using Ruby on Rails, TDD, Bootstrap. I liked that I could add a content theme to show how messages could work with people who know each other and also show my creativity to bring the app to life.

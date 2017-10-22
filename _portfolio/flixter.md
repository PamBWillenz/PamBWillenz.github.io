---
layout: post
title: Flixter
feature-img: "img/sample_feature_img.png"

thumbnail-path: "img/flixter_thumbnail.png"
short-description: Flixter -  Learn to Code!

---
[Flixter](https://flixter-pam-willenz.herokuapp.com/) is a a two sided video streaming marketplace platform, featuring credit card payment capabilities, user role management, complex user interfaces and advanced database relationships. 

<img src="/img/flixter_screenshot.png">
<br>

<h3> User's Course Show Page </h3>

<img src="/img/flixter_course_page.png">

<h3>Instructor Course Page </h3>

<img src="/img/flixter_instructor_course_page.png">
<br>

<h3>Requirements</h3>
1. App to have student and instructor accounts.
2. Once logged in, a student or instructor index page should list all classes.
3. For students, you can - 
  + sign up for a class  
  + go to class to watch a video
  + pay for it with a credit card using stripe
4. For instructors, you can -
  + change course order
  + set course price
  + add sections
  + add courses

<h3>Student Video Lesson Page </h3>

<img src="/img/flixter_video_lesson_page.png">
<br>

<h3>Student Payment Modal </h3>

<img src="/img/flixter_pay_with_stripe.png">
<br>

<h3>Implementation</h3>

Add User Authentication to the Rails App with the **Devise gem**. Go through documentation to add the necessary features you want. Generate the User model with `rails generate devise User` Rails also generates a migration file, model, spec file and routes for :users.

Add Course model, validation, controller and views
<br>
`rails generate model course`
{% highlight ruby %}
class Course < ApplicationRecord
  belongs_to :user
end
{% endhighlight %}

Add Course controllers - for students and instructors 
  + use `namespaces` to nest routes

{% highlight ruby %}
namespace :instructor do
    resources :courses, only: [:new, :create, :show]
  end
{% endhighlight %}

{% highlight ruby %} 
class Instructor::CoursesController < ApplicationController
  before_action :authenticate_user!

  def new
    @course = Course.new
  end

  def create
    @course = current_user.courses.create(course_params)
    redirect_to instructor_course_path(@course)
  end

  def show
    @course = Course.find(params[:id])
  end

  private

  def course_params
    params.require(:course).permit(:title, :description, :cost)
  end
end
{% endhighlight %}

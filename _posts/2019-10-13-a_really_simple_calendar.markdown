---
layout: post
title:      "A really simple_calendar"
date:       2019-10-13 08:37:45 -0400
permalink:  a_really_simple_calendar
---


I am creating an application where landscapers and their clients can log in and view their appointments. Appointments are best displayed on a calendar, so it was time to find a way to display a calendar in Ruby on Rails.

I found that the gem [simple_calendar](https://github.com/excid3/simple_calendar) was exactly that - a very simple way to add calendar functionality! It allows users to very quickly roll out a calendar that displays events.

If you're already using Bootstrap, simply add `gem "simple_calendar", "~> 2.0"` to your Gemfile.

Then add a model that includes a column called `start_time` with a type of `datetime` with `rails g resource Appointment name:string start_time:datetime`

Next, add the @appointments variable to controller action that will display the calendar. For example:
```
class AppointmentsController < ApplicationController
  def index
    @appointments = current_user.appointments
  end
end
```

Then add the following code where you want your calendar: 
```
<%= month_calendar events: @appointments do |date, appointments| %>
  <%= date.day %>

  <% appointments.each do |appointment| %>
    <div>
      <strong><%= appointment.start_time.strftime("%l:%M %p") %></strong><br>
      <%= link_to(appointment.landscaper.business_name, user_appointment_path(current_user, appointment), :title => "Click for more details") if user_signed_in? %>
    </div>
  <% end %>
<% end %>
```

This will display a calendar like this:
![Calendar Screenshot](https://i.imgur.com/x9DEPe8.png)



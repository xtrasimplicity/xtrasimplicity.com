---
title: "Stubbing controller methods with Capybara & Rspec"
date: 2018-08-15T09:44:50+10:00
draft: false
tags: ['rspec', 'capybara', 'ruby']
categories: ['Development', 'Ruby']
---

Although it is often a sign of code that needs to be refactored and is an anti-pattern, it is sometimes necessary to stub controller methods within RSpec when running integration/feature specs.

To avoid needing to use RSpec-mock's `allow_any_instance_of` method, which I prefer to avoid using, I tend to create a new controller instance, stub the required method against the new controller instance, and then stub the controller class' `.new` method to return the stubbed controller.

e.g:
```ruby
describe '[...]' do
  before do
    stubbed_controller = Controller.new
    user = User.first

    allow(stubbed_controller).to receive(:current_user).and_return(user)
    allow(Controller).to receive(:new).and_return(stubbed_controller)
  end
end
```
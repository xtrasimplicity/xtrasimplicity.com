---
title: "RSpec to NSpec: An introduction to testing .NET applications with NSpec for Ruby Developers"
date: 2020-08-21T17:07:22+10:00
draft: true
tags: ['rspec', 'nspec', 'c#', '.NET', 'testing']
categories: ['Development', 'Ruby', '.NET', 'C#']
---

Anyone who knows me will likely know that Ruby is my programming language of choice, but that I'm also a strong supporter of using the best tool for the job.

A recent project that I've been working on required me to consume an external, 3rd party API, so I had a choice to make - a) spend time building Ruby bindings for the external API, or b) switch to Microsoft's .NET framework to take advantage of the API provider's API client libraries. I decided to go with option `b` to simplify the API integration.

So, after installing Visual Studio Community in a Windows VM (I use Fedora as my "daily driver"), I started setting up my project as I would with Ruby - that is, adding version control, test frameworks, and other dependencies.

Arguably the biggest learning curve for me was getting my head around test-driven development (TDD) in C#. In Ruby, I would simply add `rspec` to my Gemfile, run `bundle install` and `rspec --init`, and then start writing my tests. In .NET, however, this process turned out to be very, very different.

To help others who are delving into the world of TDD on .NET for the first time, especially for those who have experience with Ruby's RSpec, I figured I'd list some of the major differences I've come across, and how I worked with them.

# Major differences
## Project Structure
In Ruby, usually your RSpec tests/specs would sit in a `specs` folder within your application's root directory, and your `spec_helper.rb` (or `rails_helper.rb` in Rails) would `require` your application.

In .NET, however, your tests actually live in a separate _project_ within the same _solution_. This project would then _reference_ your other project(s) (where your application's source-code lives).

For example, say I create a new solution called `MyApp`, I would create a `library` project for the application code, and a `console` project for the test code. 

I would then create a folder in the `Tests` project referring to the library under test, and create spec files therein which reflect the classes that I'd like to test, as shown below:

![Solution Explorer](/images/nspec-csharp-for-ruby-developers/solution.png)

To configure the `Tests` console application to run the tests, I then need to update `Program.cs` within the `Tests` application, as below:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using NSpec;
using System.IO;
using NSpec.Api;
using NSpec.Domain.Formatters;
using NSpec.Domain;

namespace Tests
{
    class Program
    {
        static void Main(string[] args)
        {
            var types = typeof(Program).Assembly.GetTypes();
            var finder = new SpecFinder(types, "");
            var tagsFilter = new Tags().Parse("");
            var builder = new ContextBuilder(finder, tagsFilter, new DefaultConventions());
            var runner = new ContextRunner(tagsFilter, new ConsoleFormatter(), false);
            var results = runner.Run(builder.Contexts().Build());

            Console.ReadLine(); // Require the user to press any key to close the app

            if (results.Failures().Count() > 0)
            {
                Environment.Exit(1);
            }
        }
    }
}
```

The next step is to add a reference to the `MyApp` library into your `Tests` application, by right clicking `References` > `Add Reference` and then ticking the `MyApp` project.

![Adding references](/images/nspec-csharp-for-ruby-developers/add_reference.png)

![Selecting the Project to reference](/images/nspec-csharp-for-ruby-developers/references_dialog.png)

To run the tests, you'll first need to set your `Tests` application as your startup project, by right clicking on the application in `Solution Explorer` and clicking `Set as Startup Project`. You can then simply start the debugger by clicking on the green arrow, or pressing `F5`.

### Mocking/Stubbing
Mocking is very, very different in .NET. `RSpec` includes some very useful mocking libraries (as part of `rspec-mocks`) "straight out of the box", whereas `NSpec` doesn't, instead leaving the developer to decide which mocking framework to use. After a little research, I settled on `Moq`.

When mocking objects in Ruby, it is common for developers to stub commands to return a double, and then perform verifications against that double. e.g.

```ruby
class App
  def get_something
  end
end

app = App.new

my_double = double

expect(app).to receive(:get_something).and_return(my_double)

expect(app.get_something).to eq(my_double)
```

In C#, with `Moq`, you would do something like this:
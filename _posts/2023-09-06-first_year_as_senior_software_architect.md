# Table of contents
* [Let's Start](#1)
* [Why Choose a Startup?](#2)
* [Mindset](#3)
* [Individual Contributor](#4)
* [Uncertainty](#5)
* [Engineering Practice](#6)
* [Predictable Delivery & Rapid prototyping](#7)
* [The end or the beginning?](#8)
* [References](#9)

## Let's Start<a name="1">
Stepping into the world of startups as a senior software architect after years of consultancy and big product company experiences was a deliberate choice I made to broaden my horizons and fuel my passion for innovation.
In this article, I will delve into the reasons behind my decision to join a startup, emphasizing the importance of a growth mindset and some engineering practices in the dynamic startup environment.

## Why Choose a Startup?<a name="2">
After immersing myself in the world of consultancy and big companies, I wanted a new challenge that would enable me to have a direct impact on a company's development. Startups offer an environment where groundbreaking ideas are supported, and where the ability to innovate and experiment is highly valued. By joining a startup, I sought to tap into a culture that encourages agility, creative problem-solving, and rapid iteration while being part of a tight-knit team working towards a shared vision.

## Mindset?<a name="3">
The Right Mindset? Growth!
Thriving in the startup ecosystem requires cultivating a growth mindset—a mindset that embraces continuous learning, adaptability, and resilience. We operate in a dynamic and ever-evolving landscape, where Change is constant and Challenges are abundant. As a senior software architect, I quickly realized that embracing a growth mindset was essential for staying ahead of the curve. This mindset fueled my curiosity, motivated me to explore and learn emerging technologies and architectural patterns, and enabled me to adapt swiftly to new requirements and business goals.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rsknrupqfbdmwcrtxgf6.jpg)

[DeNexus Inc](https://www.denexus.io/about) is a data company that wants to solve very complex problems leveraging data (tons of them).
I had no prior knowledge of DeNexus's domain, which is: quantified operational technology (OT) cyber risk. I have previous experience with OT (Operational Technology) environments, specifically in the context of discovering and managing facility assets and inventory. However, the terminology and domain concepts I encountered in my previous company were somewhat distinct.
Therefore, I had to educate myself by reading various books (see references) and seeking guidance from my more experienced colleagues. Luckily, the team consists of individuals with twenty years of OT cybersecurity and risk computation expertise.
They provided invaluable assistance in helping me understand the company's purpose and core business. For instance one of my first challenges was to revisit my studies in statistics from university in order to understand better the cyber risk quantification metrics to calculate and communicate to customers. This may appear complex, but for someone who is curious and eager to learn new things, it can be an enjoyable endeavor with a worthwhile return on investment.
As a result, after initially acquiring new knowledge, I am now able to contribute by suggesting improvements and overseeing development initiatives from an e2e perspective. This helps the company because you can scale and minimize integration and coordination between various teams.

I also quickly faced problems managing big data; starting from ingesting, cleaning, and anonymizing. The relentless pursuit of growth as an individual contributor translated into value for the company, as it drove innovation and ensured the scalability and sustainability of our software solutions.

##Individual Contributor<a name="4">
The initial team members in a startup play a pivotal role in driving its success. One of the key points is taking ownership of decisions (in my case technical and product), collaborating closely with cross-functional teams, and actively seeking opportunities to contribute beyond the role, trying to understand the business and also the domain in which the company acts.
For this reason, is important to wear multiple hats. Bridge gaps, align technical solutions with business objectives, and provide guidance to the engineering team. So if you don't want to put your hands in the jam is better to stay away :)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sskq5bzylznvtp12vwn3.png)

In a startup, the number of people is not very high, so you have to try to carry out initiatives until delivery. The scenario in which DeNexus operates is complex, as mentioned before, so we need people with different skills (engineers, cyber security, data analysts, data scientists, and compliance experts).
The engineering aspect plays a crucial role because we have to put the puzzle together and deliver something valuable to our clients. Therefore, we can't just be coders; we also need to understand when and where to intervene to find trade-offs with other team members. For example, I have had some discussions with our team of mathematical data scientists regarding finding trade-offs on the number of iterations required for our models to be executed . A mathematician will provide an optimality that can be proven mathematically, but this could mean billions of iterations, which may not be computationally feasible from an engineering perspective.  So, I started by better understanding the problem and then, together with the entire team, finding a solution that would allow us to deliver value within acceptable timeframes for our customers.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hjp6r3ndd456cjku1diw.png)

## Uncertainty<a name="5">
In the fast-paced and ever-evolving world of startups, uncertainty is a constant companion. The ability to navigate this uncertainty effectively is crucial for success.
One approach that proves invaluable in such an environment is adopting an incremental approach to problem-solving. This method emphasizes breaking down complex challenges into smaller, manageable tasks and continuously iterating and adapting based on feedback and new information.

Startups often operate in uncharted territories, disrupting industries or introducing innovative products and services. With limited resources and incomplete information, uncertainties arise at every step.

In this kind of environment, speed is of the essence. An incremental approach encourages a rapid feedback loop, enabling continuous learning and adaptation. By taking small, incremental steps, I saw we could gather feedback early and often from early adopter customers, the customer success team, and stakeholders. This feedback becomes the foundation for refining and enhancing our products, services, and strategies. Each iteration allowed us to gather new data, validate hypotheses, and adjust our course based on real-world insights.

## Engineering practice<a name="6">
In the volatile and uncertain world of a digital startup, engineering practices play a crucial role in ensuring stability, adaptability, and scalability.

We immediately started with the practice of Continuous integration, test-driven development (TDD), testing, and a DevOps approach. These helped us not only to tackle uncertainty but also contribute to managing technical debt and facilitating growth.

####Continuous Integration & Testing:
In a startup environment, where speed is one of the key points, continuous integration (CI) helps identify potential problems early, minimizing risks associated with merging conflicting code and reducing the time spent on debugging. By ensuring that code changes are continuously integrated and tested, we can maintain a stable and deployable codebase, even in the face of uncertainty. This is also very useful if you have to prototype something to quickly put in a productive environment.

Previously in DeNexus, we had a release process with some manual steps, which caused us some issues because it's normal to occasionally miss a manual step. But since we started writing, automating our tests, and automatically deploying our software, what I have noticed is that people are much more confident in making changes, trying new things quickly, and adding new features.

My two cents: test everything is like taking out insurance, it helps you when you will need it.

####Test-Driven Development (TDD):
I noticed that in the startup environment, TDD can become a valuable asset by enabling faster feedback and accelerating the development process. By writing tests up front, you can validate assumptions, identify potential flaws or gaps in logic, and iteratively refine the design of your code that is fundamental to create something easily changeable.

I also noticed that it has also the side effect to help a lot in documenting the software (you have the test and you can run them) and let people work together.

This is another key point. DeNexus is a fully remote company so we have to coordinate our work very well (USA, Switzerland, Spain, London). So, imagine a complex problem to solve with all these locations and multidisciplinary teams :)

####DevOps Approach:
DevOps, in the term of emphasize the collaboration and integration of development and operations teams.
By automating infrastructure provisioning, configuration management, and deployment pipelines, the team can rapidly iterate and adapt their software. It also helps to rapidly experiment with new services or entire solutions that could be integrated in order to help develop the product.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ti7xfx5lm9xtgiyitnco.png)

Internally in DeNexus, we have organized ourselves into chapters. We have our architectural chapter, which also includes the SRE (Site Reliability Engineering) team members (everyone can participate in it). Our goal is to establish collaboration with SRE to enable product engineering teams to be self-sufficient in infrastructure provisioning. That's why we use infrastructure as code (specifically with [Terraform](https://www.terraform.io/)), which is ultimately managed by the product engineering teams. This helps us to easily create environments for experimenting or making QA and performance tests.

For instance, I had the opportunity to set up AWS services, such as SNS (Simple Notification Service) and SQS (Simple Queue Service), with the assistance of SRE. I also automated testing in our CI (Continuous Integration) process. Every time the code is pushed into the source code repository (git) the testing pipeline automatically create infrastructure (executing terraform scripts) execute test and cleanup the environment at the end.  
I also have the independence to experiment within the DEV environment or troubleshoot by reproducing any encountered errors.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b9zfd52266xygzagw3di.png)


####Non-Overengineering:
Startups often face resource constraints, and an excessive focus on over-engineering can hinder their ability to scale. By adopting a non-over-engineering approach (make it simple not easy), you can strike a balance between building robust systems and maintaining flexibility for growth.
Non-over-engineering involves making conscious decisions about the level of complexity and sophistication of the solutions implemented, considering both the current and anticipated future needs.
This approach helps manage technical debt by preventing the accumulation of unnecessary complexity and maintaining a manageable codebase. It allows it to remain agile and adaptable, enabling the company to scale efficiently and respond to changing market dynamics without being burdened by excessive technical debt.

In DeNexus we are trying to apply the KISS principle (Keep it simple, stupid). This means that in any discussion about product increments or technical development decisions, we should always ask ourselves: do we need this right now?
We need to eliminate all unnecessary things that we don't use.
How many times have we discussed adding a column to a database that isn't used, or how much unused code do we still keep in the codebase just in case we might need it?
To avoid over-engineering, discipline is key, and we need to pay careful attention because we could fall into the opposite trap of not doing anything because nothing is needed. So, we need to be mindful of the trade-off to follow.

Remember to clean your kitchen every time you have a party :)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fs5gehaa2vbekya541dl.jpg)

This helps to have a clean and clear environment to evolve. Making software is a learning process and sometimes we need to quickly make experiments or try new technologies in order to perform some product increments. This could mess up our code base, so we need to cleanup every time we have this kind of iterations.

Having a clean and clear environment to manage allow you to make changes easily, for instance
we are reevaluating our architecture to have a more responsive environment where we can capture changes in the state of our data. For this reason, we are transitioning to an event-based architecture. We are implementing this change in parallel with the development of product features, without the need to block either one.

## Predictable Delivery & Rapid prototyping<a name="7">
Engineering practices play a pivotal role in achieving predictable delivery and enabling rapid prototyping. These practices provide startups with the necessary framework to build and iterate on their products efficiently, ultimately increasing their chances of success.

Continuous Integration and Testing help to avoid regression or too many bugs to manage in the near future. Test Driven Development (TDD), DevOps approach, and Non-over-engineering allow you to create a highly changeable codebase, this means having something that you can safely change continuously in order to create a prototype to try with your early-stage customer. Most of the time in the startup environment the team starts delivering a lot at the beginning and later on once it has accumulated a lot of technical debt it slows down rapidly. The key point here is to understand that some practices help the growth of the company itself and not the opposite. If you listen to something like: we have to go faster so avoid to do some of these practices is not because these practices slow down the development but because the people in charge of the development are not skilled to apply them.

Rapid prototyping is a fundamental practice in the startup environment, allowing for quick experimentation, validation of ideas, and gathering of user feedback.

## The end or the beginning?<a name="8">

Based on my experience I can say that the success of building a company relies on the combination of individuals with a growth mindset, technical expertise and a bit of domain knowledge.
The vision provided by the founder sets the direction and purpose of the company, but are the people involved that propels the company towards achieving that vision. This is like running a marathon, not a 100-meter sprint. So, it's necessary to pace yourself and try to make mistakes in the shortest time possible in order to learn and adjust your aim.

If you are thinking of embarking on this journey don't forget this:

> "It’s a long, hard and dirty job, but someone has to do it"

## References<a name="9">
Personal:
- [My Blog](https://dev.to/maverick198)
- [GitHub](https://github.com/sabatinim)
- [Who I am](https://www.linkedin.com/in/sabatinimarco/)
- [DeNexus Inc](https://www.denexus.io/)
- [DeNexus Blog](https://blog.denexus.io/resources)

Technical:
- [Can't be Agile without CI](https://journal.optivem.com/p/cant-be-agile-without-cicd-and-tdd#%C2%A7necessary-versus-sufficient)
- [How Measure Anything Cybersecurity Risk](https://www.amazon.com/How-Measure-Anything-Cybersecurity-Risk-ebook/dp/B0C1RJ9SR1)
- [Phoenix Project (DevOps Helping Business)](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262592)









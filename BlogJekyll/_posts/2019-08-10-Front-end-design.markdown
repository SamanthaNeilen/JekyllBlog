---
layout: post
title:  "Front-end design"
date:   2019-08-10 00:00:00 +0100

tags: UX UserInterface
---

### [Bootstrap](https://getbootstrap.com/)

When creating a new website with .NET Core, the template will already contain the Bootstrap design framework. This framework uses a grid system for creating a layout, has some out-of-the-box styled controls like dropdowns and tabs and has an icon set to use. Next to the default color scheme it has some guidance on how to create your own color schemes. Bootstrap also has an eco-system of templates based on Bootstrap to get inspiration for your own designs. 

### [Office Fabric (Core)](https://developer.microsoft.com/en-us/fabric#/get-started)

Another framework like Bootstrap is Office Fabric. This is a front-end design framework from Microsoft that can be used to design a website to look like an Office 365 application. This can be especially useful if you are creating a SharePoint hosted app or if you are creating  an app in an enterprise environment where users use a lot of the Office 365 websites. Users will get a certain feeling of consistency between all the applications and recognize iconography. The Core framework is a stylesheet that comes with a grid system, fonts, animation classes and an icon set that is used in Office 365 and contain icons for most Office applications. The website for Office Fabric also lists the colors for the Microsoft application. There are also out-of-the-box web controls, but these use the React framework. The controls that use just vanilla JavaScript have been marked deprecated. 

### [Material Design](https://material.io/design/) 

Material Design is a set of design guidelines by Google. There are several Bootstrap implementations that uses these guidelines to give you an easy access to an implementation for these guidelines to use in your application. [Use this link for an overview of a few Bootstrap components with Material Design applied](https://mdbootstrap.com/docs/jquery/components/demo/). The provided link points to an implementation that has several implementations (free or paid) using JQuery, Angular, React and Vue.

### Icon sets

When using one of the above design frameworks (or just using your own design) you may have need of some icon as visual aid for a user. The links below point to some icon sets that you could use. When using icons from different frameworks or sources be aware that, just like fonts, icons from a different set may have different line or edge shapes. Mixing and matching icons from different sets may break the feeling of consistency for your user. If you really need or want to use an icon from a different set that you are already using, consider changing al or al related current icons to the new set. By related icons I mean the ones whose functionality or concept is similar and are usually near each other on the page.

- [Font awesome icons](https://fontawesome.com/)
- [Ion icons](https://ionicons.com/) 

###  Further reading and research

The design frameworks in mentioned in this post adhere to certain style and UX (user-experience) guidelines to help you create nicer looking sites. Using these frameworks is good way of getting started but are not a sure way to create a good looking and usable site. Some links and resources to further content on what makes “good design” and “good user-experience” can be found listed below.

- [Blogpost about 8 different UX diagrams](https://www.uxbooth.com/articles/8-must-see-ux-diagrams/): The UX diagrams mentioned in this post are a good starting point for discovering some theories and guidelines for good UX.
- [The book Visual language](https://www.amazon.com/dp/9490947725/ref=cm_sw_em_r_mt_dp_U_0KNtDb84VSDWY): This book is great read if your interested in how people perceive visual information. It also has some concrete guidelines and examples on how to design a website or magazine.
- [A website about Dark patterns](https://www.darkpatterns.org/types-of-dark-pattern): This website contains a list of dark patterns for website design. Dark patterns are ways to fool or confuse a user and should be avoided.
- [The userinyerface.com game](https://userinyerface.com/): As heard on the [MS Dev Show episode talking about the Microsoft Graph web controls](https://msdevshow.com/2019/07/graph-toolkit-with-nikola-metulev/). This game challenge you on with navigating through a form with a very bad design.

And some links to podcast episodes about user interface/ user-experience design:

- [CodeNewbie: Why you should understand user interface and design with Mina Markham](https://www.codenewbie.org/podcast/why-you-should-understand-user-interface-and-design)
- [CodeNewbie: UX in healthcare with Danielle Smith](https://www.codenewbie.org/podcast/ux-in-healthcare)
- [Learn to code with me: How to break into product design with Lenora Porter](https://learntocodewith.me/podcast/product-design/)
- [DotNet Rocks: UX Design for Developers with Billy Hollis](https://dotnetrocks.com/?show=1618 )
- [DotNet Rocks: Integrating UX in your Development Process with Debbie Levitt](https://dotnetrocks.com/?show=1643)
- [Complete Developer Podcast: Dark Patterns in UI Design](https://completedeveloperpodcast.com/episode-175/)
- [Hanselminutes: Web Animation at Work with Rachel Nabors](https://hanselminutes.com/602/web-animation-at-work-with-rachel-nabors)
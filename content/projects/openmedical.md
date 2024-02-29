---
author: Thomas Cross
title: OpenMedical
date: 2023-12-19
description: Open-Source Medical portal template utilizing Bootstrap 5, React, ASP.NET Core, and Microsoft SQL Server.
tags: ["C#", "ASP.NET Core", "React", "SQL", "Microsoft SQL Server", "Typescript", ]
thumbnail: 
    url: img/openmedical.png
---

OpenMedical is a website with an emphasis on being an open-source architecture to be used for creating an open environment that can be easily expanded upon. Project has been intended for use in newly founded hospitals wanting a basic infrastructure that can be quickly expanded upon by software engineers. It uses popular, readily available frameworks with plenty of documentation to assist in usability.

OpenMedical’s structure is built utilizing a two-layered architecture utilizing an ASP.NET core web API and a React frontend utilizing typescript with a component-based architecture.

The ASP. NET core Web API was utilized as a foundation for this, using Entity Framework Core to manage database consistency and integrity, while also utilizing the built-in context functions to form our endpoints.

The frontend has been constructed using Bootstrap and React, which manages the state of the pages. Each page’s state is based on the user’s role, which is stored inside of a JWT token after login. The token contains the user’s username ID and role in order to form proper requests to the endpoint.

# Timeline of Development

{{< timeline data="openmedical-tl" background="dark" >}}
---
title: "Visual Studio 2022 Professional - Unable to Sign In Error"
date: 2025-01-24
---
Today a colleague who was using Visual Studio 2022 Professional (the latest version, 17.12.4) encountered the following error when trying to log in:

> Only loopback redirect uri is supported, but urn:ietf:wg:oauth:2.0:oob was found. Configure http://localhost or http://localhost:port both during app registration and when you create the PublicClientApplication object. See https://aka.ms/msal-net-os-browser for details

![plot](https://github.com/fabiocannas/blog/blob/main/_posts/2025-01-24-VS2022_Unable_to_Sign_In_Error/2025-01-24-VS2022_Unable_to_Sign_In_Error.png?raw=true)

After doing some research on the web, I found an [issue](https://developercommunity.visualstudio.com/t/VS2022-Professional---Unable-to-sign-in-/10795473) on the developer community, where a user was reporting the same problem.

The problem is known and Microsoft has released a fix in preview, but in the meanwhile the workaround is:

> Go to File -> Account settings -> Account Options -> on the right hand side, change the authentication mechanism to System Web Browser

I hope this post was helpful, see you in the next post :)
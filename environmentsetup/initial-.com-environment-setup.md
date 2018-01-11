Setting up a Development Environment for GTMSportswear.com
1)	Install Visual Studio 2017 with the correct components included
a.	Visual studio Professional 2017 can be found here https://www.visualstudio.com/downloads/
b.	While using the installer you will be directed to the below window. From here you can choose additional components to include with the Visual Studio core editor
c.	The following should be selected under the Workloads tab
i.	.NET desktop development
ii.	Desktop development with C++
iii.	ASP.NET and web development
iv.	Azure development
v.	.NET Core cross-platform development
d.	The below components should be added under the Individual components tab
i.	Python 3 64-bit (3.6.3)
ii.	Windows 8.1 SDK
2)	Once Visual Studio 2017 has been successfully installed a Visual Studio Team Services (VSTS) user account will need to be associated with it. The following is a way to check if your account has been created and given proper permissions (this section can be skipped). 
a.	Go to the GTM Sportswear VSTS website here https://gtmsportswear.visualstudio.com/_projects
b.	Sign in using your work email and associated password
c.	Under the settings drop down select Security 
d.	Select “GTM Sportswear – Development Group” in the drop down list on the left hand side of the page
e.	Select Members and verify that your name is included on the list of members
3)	Now Open Visual studio 2017. In the top right hand corner of Visual Studio is a prompt to sign in as a user. 
a.	Sign in using your work email and password.
b.	If your account has been given proper permissions you should be able to access GTM projects through the Team explorer window. 
c.	Select the green connection icon at the top of the Team Explorer window.
d.	Click the manage connections Icon and select Connect to a Project in the dropdown.   
e.	From here you can choose the gtm project and clone it to a destination of your choice (It is helpful to keep all your projects in one Repos folder).
4)	Next you will want to remove Gtm.Web from the list of projects to build while debugging. This is because Gtm.Web has not been configured to ignore Typescript files while building, resulting in build errors. 
a.	In Visual Studio go to Build ->>  Configuration Manager
b.	Deselect build for Gtm.Web
5)	By default, Windows may prevent Visual Studio’s emulator from displaying the website through localhost. This problem can be fixed by changing a setting through Window’s control panel.
a.	Go to Control Panel ->> Programs ->> Programs and Features ->> Turn Windows features on or off     ->> Internet Information Services
b.	Activate Internet Information Services

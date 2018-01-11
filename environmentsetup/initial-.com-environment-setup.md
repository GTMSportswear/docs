## Setting up a Development Environment for GTMSportswear.com

1.	Install Visual Studio 2017 with the correct components included.
    1.	Download Visual studio Professional 2017 here: https://www.visualstudio.com/downloads/.
    1.  The Visual Studio installer will direct you to an **Installing Visual Studio** screen. You can select additional components to include with the Visual Studio core editor. Don't deselect any components selected by default.
    1.	Make sure the following are selected under the **Workloads** tab of the **Installing Visual Studio** screen
        *	.NET desktop development
        *	Desktop development with C++
        *	ASP.NET and web development
        *	Azure development
        *	.NET Core cross-platform development
    1.	These components should be added under the **Individual Components** tab
        *	Python 3 64-bit (3.6.3)
        *	Windows 8.1 SDK
    1. Once the correct components are selected, finish the installation process. 
   
2.	Once Visual Studio 2017 has been successfully installed, a Visual Studio Team Services (VSTS) user account will need to be associated with it. The following is a way to check if your account has been created and given proper permissions. This section can be skipped if you know the correct permissions have been granted.  
    1.	Go to the GTM Sportswear VSTS website here: https://gtmsportswear.visualstudio.com/_projects.
    1.	Sign in using your work email and associated password.
    1.	Under the **settings** (looks like a gear) dropdown select **Security**. 
    1.	Select **GTM Sportswear – Development Group** in the drop down list on the left hand side of the page.
    1.	Select **Members** and verify that your name is included on the list of members.
    
3.	Now Open Visual studio 2017. In the top right hand corner of Visual Studio is a prompt to sign in as a user. 
    1.	Sign in using your work email and password.
    1.	If your account has been given proper permissions you should now be able to access GTM projects through the **Team Explorer** window. 
    1.	Select the **green connection icon** at the top of the **Team Explorer** window.
    1.	Click the **manage connections Icon** and select **Connect to a Project** in the dropdown.   
    1.	From the new view you can select the **gtm ->> gtm** project and clone it to a destination of your choice (the default location is fine).
    1.  Once the project has finished downloading, it will be automatically selected and appear in the **Solution Explorer**.
    1.  Double click on **GTMSportswear** in the **Solution Explorer**. This sets it as the default startup project.

4.	Next you will want to remove **Gtm.Web** from the list of projects to build while debugging. This is because **Gtm.Web** has not been configured to ignore Typescript files while building in Visual Studio, resulting in build errors. 
    1.	In Visual Studio go to **Build ->>  Configuration Manager** (Build is a tab along the top of Visual Studio).
    1.	Deselect build for **Gtm.Web**.

5.	By default, Windows may prevent Visual Studio’s emulator from displaying the website through localhost. This problem can be fixed by changing a setting through Window’s control panel.
    1.	Go to **Control Panel ->> Programs ->> Programs and Features ->> Turn Windows features on or off ->> Internet Information Services**.
    1.	Activate **Internet Information Services**.
    
6.  The GTMSportswear project has several web dependancies that are not tracked through source control. This is because they can be easily installed in your local environment with npm. 
    1. Open Window's command prompt and change the working directory to **\<gtm_repository\>\GTMSportswear.com**. 
    1. Run **npm install**.
    1. If Grunt is not installed, run **npm install grunt --save-dev**.
    1. Install the Grunt command line interface globally with **npm install -g grunt-cli**.
    1. Install the Karma command line interface globally with **npm install -g karma-cli**.
   
7.  Finally, check to see that the GTM website can be emulated through the Visual Studio debugger. 
    1. Run **grunt** from the command line.
    1. From Visual Studio press **F5** or alternately press the **green play button** at the top-center of Visual Studio.
    1. A locally hosted website should appear in your browser that resembles https://gtmsportswear.com/.
    

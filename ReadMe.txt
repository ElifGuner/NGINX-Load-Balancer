A minimal API is implemented which returns the argument which is given 
when starting the instances from windows powershell:

app.MapGet("/", () => args[0]);

The aim is balancing the load for 3 instances of the same service.
In the launchSettings.json file, 3 instances are defined:

    "instance1": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": false,
      "applicationUrl": "https://localhost:7250;http://localhost:5130",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "instance2": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": false,
      "applicationUrl": "https://localhost:7251;http://localhost:5131",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "instance3": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": false,
      "applicationUrl": "https://localhost:7252;http://localhost:5132",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
	
These http addresses are also used in nginx.conf files.

NGINX application should be downloaded from https://nginx.org/en/download.html.
In order to configure NGINX, downloaded files should be unzipped. 
in the ../conf/nginx.conf file, server ip/ports which will be used for load balancing
should be defined in the upstream block:

    upstream servers {
		server 127.0.0.1:5130;
		server 127.0.0.1:5131;
		server 127.0.0.1:5132;
	}
(above default definition refers to "Round Robin Algorithm". If server capacities are not same,
than "Weighted Round Robin Algorithm" can be used:

    upstream servers {
		server 127.0.0.1:5130 weight=3;
		server 127.0.0.1:5131 weight=1;
		server 127.0.0.1:5132 weight=2;
	}
)	

Than, nginx.exe should be executed. There is no interface for this application, 
so it should be controlled from task manager whether it is working.
If it is not exist in task manager, than, ../logs/error.log file should be checked.
Most of the errors are caused from the hidden characters in the nginx.conf file.
Also, some settings may be closed from Windows Security:
	Virus and threat protection
	App and browser control

TEST STEPS:
- build the minimal API successfully.
- open 3 windows powershell screens.
- execute the following commands subsequently:
  
    dotnet run instance1 -launch-profile instance1  --> first screen
	dotnet run instance2 -launch-profile instance2  --> second screen
	dotnet run instance3 -launch-profile instance3  --> third screen
	
	first instance1 is the argument that is given to the main application.
	second one is the profile that is defined in launchSettings.json file.
	
-wait for each one to be built, than execute the subsequent command.
 Otherwise, error occurs.	
-from the browser, make a request to http://localhost:8080/ (nginx server that is defined nginx.conf)
 when the request is refreshed, another instance is called every time.
 This means that requests are balanced among the instances of the service.
# How to use Powershell commands

There are 4 commands to create and handle endpoints much easier than before. While executing these commands, pattern .cs files are created automatically and are added simply to your dll file after compiling. The related functions are located in a powershell script (PCTCommands.psm1) . 

![Picture 1](./images/HowToUsePowershellCommandsPic1.JPG)

### How to use commands

   - Create a class library (.NET Framework) project

   - Add ADP.tools to project (This has not implemented as a Nuget-package yet)

   - Make sure that you added all required packages or references to your project. The essential ones are:

        - ADP.Commands

        - ADP.Logging

        - ADP.DependencyInjection

        - Network.Json

        - Ninject

        - ProconTEL-SDK

   - Open ‘Package Manager Console’ 


### Run Add-PCTEndpoint [Name]:

<!--Just enter the name of your current element right after the command (i.e. Add-PCTEndpoint MyEndpoint)-->


```
PM> Add-PCTEndpoint MyEndpoint ..... ..... Time Elapsed: 00:00:00.4718015 Endpoint class has been successfully added
```


If you need to repeat this step to rename your endpoint, please remove the Endpoint.cs from your project first. **You have to create another new class library for every single endpoint.**


![Picture 2](./images/HowToUsePowershellCommandsPic2.PNG)



Let us explain the changes you can make with an example. 'MyService'  should create an order with an order ID randomly and acts as a  subscriber. All you need to do is to put your code in  ‘SubscribeCommands’ function of 'MyService.cs' as per statement below:



```
 //MyService.cs 
 private void SubscribeCommands()    
 {      
     _commands.On("create_order", (dynamic opts) =>      
     {        
         Random rnd = new Random(DateTime.Now.Millisecond);
         string customer = opts.CustomerName;
         int orderId = rnd.Next(10000, 99999);        
         _logger.Information($"Creating order ID for the customer {customer}, New orderID {orderId}");        
         Thread.Sleep(1000);        
         _commands.Send("order_created", new { OrderId = orderId, CustomerName = customer });      
     });       
     _commands.On("create_order_sync", (dynamic opts) =>      
     {        
         Random rnd = new Random(DateTime.Now.Millisecond);        
         string customer = opts.CustomerName;        
         int orderId = rnd.Next(10000, 99999);        
         _logger.Information($"Creating order ID for the customer {customer}, New orderID {orderId}");        
         Thread.Sleep(1000);        
         return  new { OrderId = orderId, CustomerName = customer };      
     });    
 }
```



You should add some other code to Endpoint class, making sure that your  service will be initialized and loaded. In our example, it should look  like below:

```
//Endpoint.cs 
protected override void OnInitialize(DI di)    
{      
    di.UseProconTELLogger();      
    di.AddModule(new EndpointModule());      
    di.Get<MySampleClassLibraryProject.Services.MyService>().Init();    
}
```

and also add some codes to the Load() function:

```
//Endpoint.cs public override void Load()      
{        
	Bind<MySampleClassLibraryProject.Services.MyService>().ToSelf().InSingletonScope();      
}
```

### Run Add-PCTHmiendpoint [Name]

This command will create a HMI endpoint to take over the control over user interface.

```
PM> 
Add-PCTHmiEndpoint MyHmi 
..... 
..... 
Time Elapsed: 00:00:00.5152169 Endpoint class has been successfully added
```

Please be aware of adding additional packages to your project. An overview list can be found here: [List of necessary packages](List_of_packages.md) 


<!--You have to run Add-PCTService [Name] again, to add service.cs to your new HMI project. i.e.-->

<!--PM> Add-PCTService MyHmiService-->  


Now, you have to just add your code to functions as before. Let us continue our example.


```
//Endpoint.cs 
//The changes are the same as before.

//MyHmiService.cs 
//No need to change anything.
```


Since this will show a user interface, you have to create a HMI (frontend). 

![Picture 3](./images/HowToUsePowershellCommandsPic3.PNG)

Let us take a look at this with the help of an example. Remember our  order example? In order to make our endpoint taking action, we need a  trigger. It can be a simple button as below:

![Picture 4](./images/HowToUsePowershellCommandsPic4.PNG)

This view will be displayed later on, after you installed the endpoint and added it to a channel or pool. A few changes take place in your additional ViewModel.cs file (if necessary)

```
namespace MyEndpoint.Frontend.MainMenu
{
  [IsDefaultView]
  internal class MyUserControlWPFViewModel : ScreenDialogBase

  {
    private ICommandHub _commands;
    private string orderId;

    public string OrderId
    {
      get => orderId;
      private set
      {
        if (orderId == value)
          return;
        orderId = value;
        NotifyOfPropertyChange(() => OrderId);
      }
    }
    public void MainMenuViewModel(Dispatcher dispatcher, IMainViewModel mainViewModel, ICommandHub commands)
    {
      _commands = commands;
      p = (arg) => OrderId = arg.OrderId;
      _commands.On("order_created", p);
    }
    private Action<dynamic> p;


    public void CreateOrder()
    {
      var response = _commands.Get("create_order_sync", new { CustomerName = "Walczak" });
      OrderId = response.OrderId;
    }
  }
}
```

### Run Add-PCTView [Name]

```
PM> Add-PCTView MyView
....
....
....
The service MyView has been successfully added to the project 'MyProject'
```

- Open ProconTEL
- Install new created endpoint
- Enjoy your work  

If you need more information about Endpoints in general, click here : 

[How to author a custom endpoint](https://macrix.atlassian.net/wiki/spaces/PROC/pages/1762820188/How+to+author+a+custom+endpoint)
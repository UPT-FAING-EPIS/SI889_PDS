# SESION DE LABORATORIO N° 02: PATRONES DE DISEÑO ESTRUCTURALES

## OBJETIVOS
  * Comprender el funcionamiento de algunos patrones de diseño de software del tipo estructural.

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de Bash (powershell).
    - Conocimientos básicos de Contenedores (Docker).
  * Hardware:
    - Virtualization activada en el BIOS..
    - CPU SLAT-capable feature.
    - Al menos 4GB de RAM.
  * Software:
    - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior)
    - Docker Desktop 
    - Powershell versión 7.x
    - Net 6
    - Visual Studio Code

## CONSIDERACIONES INICIALES
  * Clonar el repositorio mediante git para tener los recursos necesarios

## DESARROLLO

### PARTE I: Bridge Design Pattern 

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/186e0bbd-0d14-48eb-af20-8f46dc0a08ca)

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/fab291c1-01e9-4a11-bfbd-a34609466cab)

1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o Notifications
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd Notifications
dotnet new classlib -o Notifications.Domain
dotnet sln add ./Notifications.Domain/Notifications.Domain.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new nunit -o Notifications.Domain.Tests
dotnet sln add ./Notifications.Domain.Tests/Notifications.Domain.Tests.csproj
dotnet add ./Notifications.Domain.Tests/Notifications.Domain.Tests.csproj reference ./Notifications.Domain/Notifications.Domain.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Notifications.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. Primero se necesita implementar la interfaz que servirá de PUENTE entre la clase abstracta de mensajes y las posible implementaciones de envio. Por eso en VS Code, en el proyecto Notifications.Domain proceder a crear el archivo IMessageSender.cs :
```C#
namespace Notifications.Domain
{
    public interface IMessageSender
    {
        string SendMessage(string Message);
    }
}
```
7. Ahora proceder a implementar las clases concretas o implementaiones a partir de la interfaz creada, Para esto en el proyecto Notifications.Domain proceder a crear los archivos siguientes:
> SmsMessageSender.cs
```C#
namespace Notifications.Domain
{
    public class SmsMessageSender : IMessageSender
    {
        public string SendMessage(string Message)
        {
            return "'" + Message + "' : This Message has been sent using SMS";
        }
    }
}
```
> EmailMessageSender.cs
```C#
namespace Notifications.Domain
{
    public class EmailMessageSender : IMessageSender
    {
        public string SendMessage(string Message)
        {
            return "'" + Message + "'   : This Message has been sent using Email";
        }
    }
}
```
8. Seguidamente crear la clase abstracta que permitira definir los posibles tipos de mensajes por lo que en el proyecto de Notifications.Domain se debe agregar el archivo AbstractMessage.cs con el siguiente código:
```C#
namespace Notifications.Domain
{
    public abstract class AbstractMessage
    {
        protected IMessageSender _messageSender;
        public abstract string SendMessage(string Message);        
    }
}
```
9. Sobre esta clase abstracta ahora se necesita implementar los tipos de mensajes concretos, para eso adicionar los siguientes archivos al proyecto Notifications.Domain:
> ShortMessage.cs
```C#
namespace Notifications.Domain
{
    public class ShortMessage: AbstractMessage
    {
        public const string LARGE_ERROR_MESSAGE = "Unable to send the message as length > 10 characters";
        public ShortMessage(IMessageSender messageSender)
        {
            this._messageSender = messageSender;
        }
        public override string SendMessage(string Message)
        {
            if (Message.Length <= 25)
                return _messageSender.SendMessage(Message);
            else
                throw new ArgumentException(LARGE_ERROR_MESSAGE);
        }
    }
}
```
> LongMessage.cs
```C#
namespace Notifications.Domain
{
    public class LongMessage: AbstractMessage
    {
        public LongMessage(IMessageSender messageSender)
        {
            this._messageSender = messageSender;
        }
        public override string SendMessage(string Message)
        {
           return _messageSender.SendMessage(Message);
        }
    }
}
```
10. Ahora proceder a implementar unas pruebas para verificar el correcto funcionamiento de la aplicación. Para esto al proyecto Notifications.Domain.Tests adicionar el archivo MessageTests.cs y agregar el siguiente código:
```C#
using Notifications.Domain;
using NUnit.Framework;

namespace Notifications.Domain.Tests
{
    public class MessageTests
    {
        [Test]
        public void GivenLongMessage_WhenSend_ThenEmailIsTriggered()
        {
            string Message = "Este es un mensaje bien pero bien largoooooooooooooooooooooooo.";
            AbstractMessage longMessage = new LongMessage(new EmailMessageSender());
            var confirm = longMessage.SendMessage(Message);
            Assert.IsTrue(!string.IsNullOrEmpty(confirm));
            Assert.IsTrue(confirm.Contains(Message));
        }
        [Test]
        public void GivenShortMessage_WhenSend_ThenSMSIsTriggered()
        {
            string Message = "Este es un mensaje corto.";
            AbstractMessage shortMessage = new ShortMessage(new SmsMessageSender());
            var confirm = shortMessage.SendMessage(Message);
            Assert.IsTrue(!string.IsNullOrEmpty(confirm));
            Assert.IsTrue(confirm.Contains(Message));
        }
        [Test]
        public void GivenLargeMessage_WhenSendinSMS_ThenOccursException()
        {
            string Message = "Este es un mensaje largooooooooooooooooo.";
            AbstractMessage shortMessage = new ShortMessage(new SmsMessageSender());
            Assert.Throws<ArgumentException>(
                () => shortMessage.SendMessage(Message)
                , ShortMessage.LARGE_ERROR_MESSAGE);
        }
    }
}
```
11. Ahora necesitamos comprobar las pruebas contruidas para eso abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar los comandos:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
12. Si las pruebas se ejecutaron correctamente debera aparcer un resultado similar al siguiente:
```Bash
Passed!  - Failed:     0, Passed:     3, Skipped:     0, Total:     3, Duration: 5 ms
```


### PARTE II: Facade Design Pattern

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/ece5c02f-fe5e-4125-91f4-7479f6c3d746)


1. Iniciar una nueva instancia de la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o CustomerApp
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd CustomerApp
dotnet new classlib -o CustomerApp.Domain
dotnet sln add ./CustomerApp.Domain/CustomerApp.Domain.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new nunit -o CustomerApp.Domain.Tests
dotnet sln add ./CustomerApp.Domain.Tests/CustomerApp.Domain.Tests.csproj
dotnet add ./CustomerApp.Domain.Tests/CustomerApp.Domain.Tests.csproj reference ./CustomerApp.Domain/CustomerApp.Domain.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto CustomerApp.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. Primero se necesita implementar la entidad Cliente, para esto crear el archivo Customer.cs en el proyecto CustomerApp.Domain con el siguiente código:
```C#
namespace CustomerApp.Domain
{
    public class Customer
    {
        public string Name { get; set; }
        public string Email { get; set; }
        public string MobileNumber { get; set; }
        public string Address { get; set; }
        public string Password { get; set; }
        public static Customer Create(string name, string email, string mobileNumber, string address, string password)
        {
            return new Customer() {
                Name = name, Email = email, MobileNumber = mobileNumber, Address = address, Password = password
            };
        }
    }
}
```
7. Ahora se debe implementar cada una de clases correspondiente al flujo de creaciòn del cliente (validar, guardar y enviar email) para eso se deberan crear los siguientes archivos con el còdigo correspondiente:
> Validator.cs
```C#
namespace CustomerApp.Domain
{
    public class Validator
    {
        public bool ValidateCustomer(Customer customer)
        {
            //Need to Validate the Customer Object
            if (string.IsNullOrEmpty(customer.Name)) throw new ArgumentException("Name can't be null or empty");
            if (string.IsNullOrEmpty(customer.Email)) throw new ArgumentException("Email can't be null or empty");
            if (string.IsNullOrEmpty(customer.MobileNumber)) throw new ArgumentException("MobileNumber can't be null or empty");
            if (string.IsNullOrEmpty(customer.Address)) throw new ArgumentException("Address can't be null or empty");
            return true;
        }
    }
}
```
> DataAccessLayer.cs
```C#
namespace CustomerApp.Domain
{
    public class DataAccessLayer
    {
        public List<Customer> Customers { get; set; }
        public DataAccessLayer()
        {
            Customers = new List<Customer>();
        }
        public bool SaveCustomer(Customer customer)
        {
            Customers.Add(customer);
            return true;
        }        
    }
}
```
> EmailService.cs
```C#
using System.Net;
using System.Net.Mail;

namespace CustomerApp.Domain
{
    public class EmailService
    {
        public bool SendRegistrationEmail(Customer customer)
        {
            var smtpClient = new SmtpClient("smtp.gmail.com")
            {
                UseDefaultCredentials = false,
                //Port = 587,
                Credentials = new NetworkCredential(customer.Email, customer.Password),
                EnableSsl = true,
            };
            var mailMessage = new MailMessage
            {
                From = new MailAddress(customer.Email),
                Subject = "Test mail",
                Body = "<h1>Hello</h1>",
                IsBodyHtml = true,
            };
            mailMessage.To.Add(customer.Email);
            //smtpClient.Send(mailMessage);
            return true;
        }        
    }
}
```

8. Para probar esta implementación, crear el archivo CustomerTests.cs en el proyecto CustomerApp.Domain.Tests:
```C#
using NUnit.Framework;

namespace CustomerApp.Domain.Tests
{
    public class CustomerTests
    {
        [Test]
        public void GivenANewCustomer_WhenRegister_ThenIsValidatedSavedEmailedSuccessfully()
        {
            //Step1: Create an Instance of Customer Class
            Customer customer = Customer.Create(
                "Jose Cuadros","p.cuadros@gmail.com","1234567890","Tacnamandapio","str0ng.pa55");
            
            //Step2: Validate the Customer
            Validator validator = new Validator();
            bool IsValid = validator.ValidateCustomer(customer);
            //Step3: Save the Customer Object into the database
            DataAccessLayer dataAccessLayer = new DataAccessLayer();
            bool IsSaved = dataAccessLayer.SaveCustomer(customer);
            //Step4: Send the Registration Email to the Customer
            EmailService email = new EmailService();
            bool IsEmailed = email.SendRegistrationEmail(customer);
            
            Assert.IsNotNull(customer);
            Assert.Greater(dataAccessLayer.Customers.Count,0);
            Assert.IsTrue(IsValid);
            Assert.IsTrue(IsSaved);
            Assert.IsTrue(IsEmailed);
        }
    }
}
```
9. Ahora necesitamos comprobar las pruebas contruidas para eso abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar el comando:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
10. Si las pruebas se ejecutaron correctamente debera aparcer un resultado similar al siguiente:
```Bash
Total tests: 1. Passed: 1. Failed: 0. Skipped: 0
```
11. Entonces ¿cuál es problema con este diseño? Funciona.... pero el problema es que ahora existen muchos sub sistemas como Validador, Acceso a Datos y Servicio de Email y el cliente que las utilice necesita seguir la secuencia apropiada para crear y consumir los objetos de los subsistemas. Existe una posibilidad que el cliente no siga esta secuencia apropiada o que olvide incluir o utilizar alguno de estos sub sistemas. Entonces si en vez de darle acceso a los sub sistemas, se crea una sola interfaz y se le brinda acceso al cliente para realizar el registo, asi la lógica compleja se traslada a esta interfaz sencilla. Para esto se utilizará el patrón FACHADA el cual escondera toda la complejidad y brindará un solo metodo cimple de usar al cliente.

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/a9cb73bb-c996-4e9a-bf4c-f665f1957119)

12. Para lo cual proceder a crear el archivo CustomerRegistration.cs en el proyecto CustomerApp.Domain, con el siguiente contenido:
```C#
using CustomerApp.Domain;

public class CustomerRegistration
{
    public bool RegisterCustomer(Customer customer)
    {
        //Step1: Validate the Customer
        Validator validator = new Validator();
        bool IsValid = validator.ValidateCustomer(customer);
        //Step1: Save the Customer Object into the database
        DataAccessLayer customerDataAccessLayer = new DataAccessLayer();
        bool IsSaved = customerDataAccessLayer.SaveCustomer(customer);
        //Step3: Send the Registration Email to the Customer
        EmailService email = new EmailService();
        email.SendRegistrationEmail(customer);
        return true;
    }
}
```
8. Finalmente adciionar un nuevo método de prueba en la clase CustomerTests para comprobar el funcionamiento de la nueva clase creada:
```C#
        [Test]
        public void GivenANewCustomer_WhenRegister_ThenIsRegisteredSuccessfully()
        {
            //Step1: Create an Instance of Customer Class
            Customer customer = Customer.Create(
                "Jose Cuadros","p.cuadros@gmail.com","1234567890","Tacnamandapio","str0ng.pa55");
            //Step2: Using Facade Class
            bool IsRegistered = new CustomerRegistration().RegisterCustomer(customer);
            Assert.IsNotNull(customer);
            Assert.IsTrue(IsRegistered);
        }     
```
9. Ahora necesitamos comprobar las pruebas contruidas para eso abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar el comando:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
10. Si las pruebas se ejecutaron correctamente debera aparcer un resultado similar al siguiente:
```Bash
Passed!  - Failed:     0, Passed:     2, Skipped:     0, Total:     2, Duration: 11 ms 
```

---
## Actividades Encargadas
1. Crear un nuevo proyecto de dominio y su respectivo proyecto de pruebas utilizando otro patrón de diseño ESTRUCTURAL.

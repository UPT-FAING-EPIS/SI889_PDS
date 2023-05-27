# SESION DE LABORATORIO N° 03: PATRONES DE DISEÑO DE COMPORTAMIENTO

## OBJETIVOS
  * Comprender el funcionamiento de algunos patrones de diseño de software del tipo de comportamiento.

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de Bash (powershell).
    - Conocimientos básicos de C# y Visual Studio Code.
  * Hardware:
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

### PARTE I: Strategy Design Pattern 

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/21cf440e-8156-498c-afd7-95c066ffaa93)
En la imagen Steve compra un monitor y una lavadora, pero a la hora de acercarse a la ventanilla existen tres formas de pagar: Tarjeta de Crédito, Tarjeta de Débito y Efectivo.

1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o Payment
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd Payment
dotnet new classlib -o Payment.Domain
dotnet sln add ./Payment.Domain/Payment.Domain.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new nunit -o Payment.Domain.Tests
dotnet sln add ./Payment.Domain.Tests/Payment.Domain.Tests.csproj
dotnet add ./Payment.Domain.Tests/Payment.Domain.Tests.csproj reference ./Payment.Domain/Payment.Domain.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Payment.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. Primero se necesita implementar la interfaz que servirá de ESTRATEGIA base para las posibles implementaciones de pagos. Por eso en VS Code, en el proyecto Notifications.Domain proceder a crear el archivo IPaymentStrategy.cs :
```C#
namespace Payment.Domain
{
    public interface IPaymentStrategy
    {
        bool Pay(double amount);
    }
}
```
7. Ahora proceder a implementar las clases concretas o implementaciones a partir de la interfaz creada, Para esto en el proyecto Payment.Domain proceder a crear los archivos siguientes:
> CreditCardPaymentStrategy.cs
```C#
namespace Payment.Domain
{
    public class CreditCardPaymentStrategy : IPaymentStrategy
    {
        public bool Pay(double amount)
        {
            Console.WriteLine("Customer pays Rs " + amount + " using Credit Card");
            return true;
        }
    }
}
```
> DebitCardPaymentStrategy.cs
```C#
namespace Payment.Domain
{
    public class DebitCardPaymentStrategy : IPaymentStrategy
    {
        public bool Pay(double amount)
        {
            Console.WriteLine("Customer pays Rs " + amount + " using Debit Card");
            return true;
        }
    }
}
```
> CashPaymentStrategy.cs
```C#
namespace Payment.Domain
{
    public class CashPaymentStrategy : IPaymentStrategy
    {
        public bool Pay(double amount)
        {
            Console.WriteLine("Customer pays Rs " + amount + " By Cash");
            return true;
        }
    }
}
```
8. Seguidamente crear la clase que funcionara de contexto y permitira la ejecución de cualquier estrategia, por lo que en el proyecto de Payment.Domain se debe agregar el archivo PaymentContext.cs con el siguiente código:
```C#
namespace Payment.Domain
{
    public class PaymentContext
    {
        // The Context has a reference to the Strategy object.
        // The Context does not know the concrete class of a strategy. 
        // It should work with all strategies via the Strategy interface.
        private IPaymentStrategy PaymentStrategy;
        // The Client will set what PaymentStrategy to use by calling this method at runtime
        public void SetPaymentStrategy(IPaymentStrategy strategy)
        {
            PaymentStrategy = strategy;
        }
        // The Context delegates the work to the Strategy object instead of
        // implementing multiple versions of the algorithm on its own.
        public bool Pay(double amount)
        {
            return PaymentStrategy.Pay(amount);
        }
    }
}
```
9. Adicionalmente para facilitar la utilización de las diferentes estrategias adicionaremos una fachada, para eso crear el archivo PaymentService.cs en el proyecto Payment.Domain:
```C#
namespace Payment.Domain
{
    public class PaymentService
    {
        public bool ProcessPayment(int SelectedPaymentType, double Amount)
        {
            //Create an Instance of the PaymentContext class
            PaymentContext context = new PaymentContext();
            if (SelectedPaymentType == (int)PaymentType.CreditCard)
            {
                context.SetPaymentStrategy(new CreditCardPaymentStrategy());
            }
            else if (SelectedPaymentType == (int)PaymentType.DebitCard)
            {
                context.SetPaymentStrategy(new DebitCardPaymentStrategy());
            }
            else if (SelectedPaymentType == (int)PaymentType.Cash)
            {
                context.SetPaymentStrategy(new CashPaymentStrategy());
            }
            else
            {
                throw new ArgumentException("You Select an Invalid Payment Option");
            }
            //Finally, call the Pay Method
            return context.Pay(Amount);;
        }
    }
    public enum PaymentType
    {
        CreditCard = 1,  // 1 for CreditCard
        DebitCard = 2,   // 2 for DebitCard
        Cash = 3, // 3 for Cash
    }
}
```
10. Ahora proceder a implementar unas pruebas para verificar el correcto funcionamiento de la aplicación. Para esto al proyecto Payment.Domain.Tests adicionar el archivo PaymentTests.cs y agregar el siguiente código:
```C#
using System;
using NUnit.Framework;
using Payment.Domain;
namespace Payment.Domain.Tests
{
    public class PaymentTests
    {
        [TestCase(1, 1000)]
        [TestCase(2, 2000)]
        [TestCase(3, 3000)]
        public void GivenAValidPaymentTypeAndAmount_WhenProcessPayment_ResultIsSuccesful(int paymentType, double amount)
        {
            bool PaymentResult = new PaymentService().ProcessPayment(paymentType, amount);
            Assert.IsTrue(PaymentResult);
        }
        [TestCase(4, 4000)]
        public void GivenAnUnknownPaymentTypeAndAmount_WhenProcessPayment_ResultIsError(int paymentType, double amount)
        {
            //bool PaymentResult = new PaymentService().ProcessPayment(paymentType, amount);
            var ex = Assert.Throws<ArgumentException>(
                () => new PaymentService().ProcessPayment(paymentType, amount));
            Assert.That(ex.Message, Is.EqualTo("You Select an Invalid Payment Option"));
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
Passed!  - Failed:     0, Passed:     4, Skipped:     0, Total:     4, Duration: 12 ms
```

13. Finalmente se puede apreciar que existen tres componentes principales en el patrón ESTARTEGIA:
a. Estrategia: declarada en una interfac para ser implementada para todos los algoritmos soportado
b. EstrategiaConcreta: Es la implementa la estrategia para cada algoritmo
c. Conexto: esta es la clase que mantiene la referencia al objeto Estrategia y luego utiliza la referencia para llamar al algoritmo definido por cada EstrtaegiaConcreta

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/e132b3dd-1b5d-4cdf-a91d-fe114071c4bb)


### PARTE II: Command Design Pattern

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/ece5c02f-fe5e-4125-91f4-7479f6c3d746)


1. Iniciar una nueva instancia de la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o ATM
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd CustomerApp
dotnet new classlib -o ATM.Domain
dotnet sln add ./ATM.Domain/ATM.Domain.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new nunit -o ATM.Domain.Tests
dotnet sln add ./ATM.Domain.Tests/ATM.Domain.Tests.csproj
dotnet add ./ATM.Domain.Tests/ATM.Domain.Tests.csproj reference ./ATM.Domain/ATM.Domain.csproj
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

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
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Notifications.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

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
    }
}
```
7. Ahora implementar el validador de datos del cliente

```C#
namespace Bank.Domain
{
    public abstract class CreditCardFactoryMethod
    {
        protected abstract ICreditCard MakeProduct();
        // Also note that The Creator's primary responsibility is not creating products. 
        // Usually, it contains some core business logic that relies on Product objects, returned by the factory method. 
        public ICreditCard CreateProduct()
        {
            //Call the MakeProduct which will create and return the appropriate object 
            ICreditCard creditCard = this.MakeProduct();
            //Return the Object to the Client
            return creditCard;
        }
    }
}
```
2. Ahora proceder a crear las implementaciones de la clase abstracta anterior para cada producto, crear los siguientes archivos en el proyecto Bank.Domain:
> MoneyBackFactoryMethod.cs
```C#
namespace Bank.Domain
{
    public class MoneyBackFactoryMethod : CreditCardFactoryMethod
    {
        protected override ICreditCard MakeProduct()
        {
            ICreditCard product = new MoneyBack();
            return product;
        }
    }
}
```
> PlatinumFactoryMethod.cs
```C#
namespace Bank.Domain
{
    public class PlatinumFactoryMethod: CreditCardFactoryMethod
    {
        protected override ICreditCard MakeProduct()
        {
            ICreditCard product = new Platinum();
            return product;
        }
    }
}
```
> TitaniumFactoryMethod.cs
```C#
namespace Bank.Domain
{
    public class TitaniumFactoryMethod : CreditCardFactoryMethod
    {
        protected override ICreditCard MakeProduct()
        {
            ICreditCard product = new Titanium();
            return product;
        }
    }
}
```
3. Para probar esta implementacón, modificar la clase de pruebas CreditCardTests y adicionar los siguientes métodos:
```C#
        [Test]
        public void GivenCreditTypePlatinumChoosen_WhenRequestCreditCard_ThenNewValidCreditCard()
        {
            ICreditCard creditCard = new PlatinumFactoryMethod().CreateProduct();
            Assert.IsNotNull(creditCard);
            Assert.IsNotEmpty(creditCard.GetCardType());
            Assert.GreaterOrEqual(creditCard.GetCreditLimit(), 0);
            Assert.GreaterOrEqual(creditCard.GetAnnualCharge(), 0);
        }

        [Test]
        public void GivenCreditTypeTitaniumChoosen_WhenRequestCreditCard_ThenNewValidCreditCard()
        {
            ICreditCard creditCard = new TitaniumFactoryMethod().CreateProduct();
            Assert.IsNotNull(creditCard);
            Assert.IsNotEmpty(creditCard.GetCardType());
            Assert.AreEqual(creditCard.GetCardType(),"Titanium Edge");
            Assert.GreaterOrEqual(creditCard.GetCreditLimit(), 0);
            Assert.GreaterOrEqual(creditCard.GetAnnualCharge(), 0);
        }
```
4. Ejecutar nuevamente el paso 9 (Parte I) para lo cual se obtendra una respuesta similar a la siguiente:
```Bash
Passed!  - Failed:     0, Passed:     3, Skipped:     0, Total:     3, Duration: 9 ms
```
5. Finalmente podemos confirmar con este patròn un desacoplamiento de la clase que lo ejecuta, asimismo la reglas de creación ya no dependen de las clausula IF-ELSE, por lo que para crear un nuevo tipo de tarjeta solo será necesario crear una nueva clase basada en la clase abstracta de CreditCardFactoryMethod:

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/bbad4ef3-4f18-4db3-85c0-c7f4f28e5ef0)

---
## Actividades Encargadas
1. Crear un nuevo proyecto de dominio y su respectivo proyecto de pruebas utilizando otro patrón de diseño CREACIONAL.

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

6. En VS Code, en el proyecto Notifications.Domain proceder a crear el archivo IMessageSender.cs e introducir el siguiente código:
```C#
namespace Notifications.Domain
{
    public interface IMessageSender
    {
        void SendMessage(string Message);
    }
}
```
7. En el proyecto Bank.Domain proceder a crear las implementaciones de a interfaz creada en el paso previo para eso añadimos los archivos:
> MoneyBack.cs
```C#
namespace Bank.Domain
{
    public class MoneyBack : ICreditCard
    {
        public string GetCardType()
        {
            return "MoneyBack";
        }
        public int GetCreditLimit()
        {
            return 15000;
        }
        public int GetAnnualCharge()
        {
            return 500;
        }
    }
}
```
> Platinum.cs
```C#
namespace Bank.Domain
{
    public class Platinum : ICreditCard
    {
        public string GetCardType()
        {
            return "Platinum Plus";
        }
        public int GetCreditLimit()
        {
            return 35000;
        }
        public int GetAnnualCharge()
        {
            return 2000;
        }
    }
}
```
> Titanium.cs
```C#
namespace Bank.Domain
{
    public class Titanium : ICreditCard
    {
        public string GetCardType()
        {
            return "Titanium Edge";
        }
        public int GetCreditLimit()
        {
            return 25000;
        }
        public int GetAnnualCharge()
        {
            return 1500;
        }
    }
}
```
8. Luego en el proyecto Bank.Domain.Tests añadir un nuevo archivo CreditCardTests.cs e introducir el siguiente código:
```C#
using Bank.Domain;
using NUnit.Framework;

namespace Bank.Domain.Tests
{
    public class CreditCardTests
    {
        [Test]
        public void GivenCreditTypeSelected_WhenRequestCreditCard_ThenNewValidCreditCard()
        {
            string cardType = "MoneyBack";
            ICreditCard? cardDetails = null;
            if (cardType == "MoneyBack")
            {
                cardDetails = new MoneyBack();
            }
            else if (cardType == "Titanium")
            {
                cardDetails = new Titanium();
            }
            else if (cardType == "Platinum")
            {
                cardDetails = new Platinum();
            }

            Assert.IsNotNull(cardDetails);
            Assert.IsNotEmpty(cardDetails.GetCardType());
            Assert.GreaterOrEqual(cardDetails.GetCreditLimit(), 0);
            Assert.GreaterOrEqual(cardDetails.GetAnnualCharge(), 0);
        }
    }
}
```
9. Ahora necesitamos comprobar las pruebas contruidas para eso abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar los comandos:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
10. Si las pruebas se ejecutaron correctamente debera aparcer un resultado similar al siguiente:
```Bash
Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: 5 ms
```
11. Funciona pero ¿es correcta la implementación del código? ¿Qué problemas tiene esta implementación?
* Primero tenemos un problema de Alto Acoplamiento entre la clase de prueba y las clases productos (MoneyBack, Titanium y Platinum). Asi que cuando hay un cambio en una d elas clases todas las demàs deberan ser cambiadas.
* Segundo, si se adiciona un nuevo tipo de tarjeta de crédito, necesitamos hacer cambios en la lògica de creación que se encuentra en el metod de prueba, adicionando una nueva condición IF-ELSE lo cual no solo complica el desarrollo, sino también el proceso pruebas.
  
12. Para solucionar los problemas anteriores mencionados utilizaremos el patrón de diseño FABRICA, para lo cual ahora en el proyecto Bank.Domain proceder a agregar el archivo CreditCarFactory.cs con el siguiente código:
```C#
namespace Bank.Domain
{
    public class CreditCardFactory
    {
        public static ICreditCard GetCreditCard(string cardType)
        {
            ICreditCard? cardDetails = null;
            if (cardType == "MoneyBack")
            {
                cardDetails = new MoneyBack();
            }
            else if (cardType == "Titanium")
            {
                cardDetails = new Titanium();
            }
            else if (cardType == "Platinum")
            {
                cardDetails = new Platinum();
            }
            return cardDetails; 
        }
    }
}
```
13. Adicionalmente modificar la clase de pruebas CreditCardTests, con el siguiente código:
```C#
using Bank.Domain;
using NUnit.Framework;

namespace Bank.Domain.Tests
{
    public class CreditCardTests
    {
        [Test]
        public void GivenCreditTypeSelected_WhenRequestCreditCard_ThenNewValidCreditCard()
        {
            string cardType = "MoneyBack";
            ICreditCard? cardDetails = CreditCardFactory.GetCreditCard(cardType);
            Assert.IsNotNull(cardDetails);
            Assert.IsNotEmpty(cardDetails.GetCardType());
            Assert.GreaterOrEqual(cardDetails.GetCreditLimit(), 0);
            Assert.GreaterOrEqual(cardDetails.GetAnnualCharge(), 0);
        }
    }
}
```
14. Al ejecutar nuevamente el paso 9 deberia seguir funcionando correctamente.

15. Con esto se aplicado el patrón de diseño FABRICA de la siguiente manera:
![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/bae74678-32e7-454f-96dc-bf4f357c676c)

> Pero con este patrón se ha solucionado parcialmente los problemas indicados en el punto 11, en especifico solo se ha reducido en cierto porcentaje el Alto Acoplamiento.
https://dotnettutorials.net/lesson/factory-method-design-pattern-csharp/

### PARTE II: Factory Method Design Pattern

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/09109954-6f0f-4db3-8449-c82b4abcfa4d)

1. Utilizando el proyecto de la primera parte proceder a crear el archivo CreditCardAbstractMethod.cs en el proyecto Bank.Domain

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

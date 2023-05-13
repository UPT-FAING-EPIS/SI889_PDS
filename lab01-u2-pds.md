# SESION DE LABORATORIO N° 01: PATRONES DE DISEÑO CREACIONALES

## OBJETIVOS
  * Comprender el funcionamiento de algunos patrones de diseño de software del tipo creacional.

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

### PARTE I: Factory Design Pattern

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/f810f99d-efe2-4ec8-a04c-83dee3872787)

1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o Bank
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd Bank
dotnet new classlib -o Bank.Domain
dotnet sln add ./Bank.Domain/Bank.Domain.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new nunit -o Bank.Domain.Tests
dotnet sln add ./Bank.Domain.Tests/Bank.Domain.Tests.csproj
dotnet add ./Bank.Domain.Tests/Bank.Domain.Tests.csproj reference ./Bank.Domain/Bank.Domain.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Bank.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. En VS Code, en el proyecto Bank.Domain proceder a crear el archivo ICreditCard.cs e introducir el siguiente código:
```C#
namespace Bank.Domain
{
    public interface ICreditCard
    {
        string GetCardType();
        int GetCreditLimit();
        int GetAnnualCharge();
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

14. Ahora es tiempo de mejorar el código y refactorizar, para lo cual modificar la clase BankAccount de la siguiente manera:
```C#
using System;
namespace Bank.Domain
{
    public class BankAccount
    {
        public const string DebitAmountExceedsBalanceMessage = "Debit amount exceeds balance";
        public const string DebitAmountLessThanZeroMessage = "Debit amount is less than zero";
        private readonly string m_customerName;
        private double m_balance;
        private BankAccount() { }
        public BankAccount(string customerName, double balance)
        {
            m_customerName = customerName;
            m_balance = balance;
        }
        public string CustomerName { get { return m_customerName; } }
        public double Balance { get { return m_balance; }  }
        public void Debit(double amount)
        {
            if (amount > m_balance)
                throw new ArgumentOutOfRangeException("amount", amount, DebitAmountExceedsBalanceMessage);
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount", amount, DebitAmountLessThanZeroMessage);
            m_balance -= amount; 
        }
        public void Credit(double amount)
        {
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance += amount;
        }
    }
}
```
15. Adicionar el siguiente método de prueba en la clase BankAccountTests, que permitira verificar si el monto de retiro es mayor al saldo de la cuenta.
```C#
        [Test]
        public void Debit_WhenAmountIsMoreThanBalance_ShouldThrowArgumentOutOfRange()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 20.0;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);
            // Act
            try
            {
                account.Debit(debitAmount);
            }
            catch (System.ArgumentOutOfRangeException e)
            {
                // Assert
                StringAssert.Contains(BankAccount.DebitAmountExceedsBalanceMessage, e.Message);
            }
        }
```
16. Volver a ejecutar el paso 8 y verificar el resultado, debería ser similar a lo siguiente
```
Passed!  - Failed:     0, Passed:     3, Skipped:     0, Total:     3, Duration: 12 ms
```
17. Finalmente proceder a verificar la cobertura, dentro del proyecto Primes.Tests se dede haber generado una carpeta o directorio TestResults, en el cual posiblemente exista otra subpcarpeta o subdirectorio conteniendo un archivo con nombre `coverage.cobertura.xml`, si existe ese archivo proceder a ejecutar los siguientes comandos desde la linea de comandos abierta anteriomente, de los contrario revisar el paso 8:
```
dotnet tool install -g dotnet-reportgenerator-globaltool
ReportGenerator "-reports:./*/*/*/coverage.cobertura.xml" "-targetdir:Cobertura" -reporttypes:HTML
```
18. El comando anterior primero proceda instalar una herramienta llamada ReportGenerator (https://reportgenerator.io/) la cual mediante la segunda parte del comando permitira generar un reporte en formato HTML con la cobertura obtenida de la ejecución de las pruebas. Este reporte debe localizarse dentro de una carpeta llamada Cobertura y puede acceder a el abriendo con un navegador de internet el archivo index.htm.

---
## Actividades Encargadas
1. Adicionar los escenarios, casos de prueba, metodos de prueba y modificaciones para verificar el método de crédito.

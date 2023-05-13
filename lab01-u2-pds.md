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

### PARTE I: Factory Method Design Pattern

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
6. En el proyecto Bank.Domain proceder a crear las implementaciones de a interfaz creada en el paso previo para eso añadimos los archivos:
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
7. Luego en el proyecto Bank.Domain.Tests añadir un nuevo archivo CreditCardTests.cs e introducir el siguiente código:
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
8. Abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar los comandos:
```Bash
dotnet test --collect:"XPlat Code Coverage"
```
9. El paso anterior debe producir un error por lo que sera necesario escribir el código mecesario para que el test funcione. 
```Bash
Failed!  - Failed:     1, Passed:     0, Skipped:     0, Total:     1, Duration: < 1 ms
```
10. En el proyecto Bank.Domain, modificar la clase BankAccount, en la linea 21 con el siguiente contenido:
```C#
       m_balance -= amount;
```
11. Ejecutar nuevamente el pase 8 y ahora deberia devolver algo similar a lo siguiente:
```Bash
Passed!  - Failed:     0, Passed:     1, Skipped:     0, Total:     1, Duration: < 1 ms
```
12. Con la finalidad de aumentar la confienza en la aplicación, se ampliará el rango de pruebas para lo cual editar la clase de prueba BankAccountTests y adicionar el método siguiente que contempla un escenario de prueba diferente:
```C#
        [Test]
        public void Debit_WhenAmountIsLessThanZero_ShouldThrowArgumentOutOfRange()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = -100.00;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);
            // Act and assert
            Assert.Throws<System.ArgumentOutOfRangeException>(() => account.Debit(debitAmount));
        }
```
13. Ejecutar nuevamente el paso 8 para lo cual se obtendra una respuesta similar a la siguiente:
```
Passed!  - Failed:     0, Passed:     2, Skipped:     0, Total:     2, Duration: 9 ms
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

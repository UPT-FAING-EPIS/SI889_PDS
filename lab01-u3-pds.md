# SESION DE LABORATORIO N° 01: MODELO VISTA VISTA-MODELO (MVVM)

## OBJETIVOS
  * Comprender el funcionamiento del patrón de presentación Modelo Vista Vista-Modelo.

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

### PARTE I: Creción del proyecto

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/f810f99d-efe2-4ec8-a04c-83dee3872787)

1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva aplicación WPF.
```
dotnet new wpf -o ClientUI
```
3. Acceder a la solución creada.
```
cd ClientUI
```
4. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. 

### PARTE II: Creación del modelo

5. En VS Code, en el proyecto ClientUI crear una carpeta `models` y proceder a crear el archivo ClientDto.cs que funcionara como Modelo base para el traslado de datos e introducir el siguiente código:
```C#
namespace ClientUI.models
{
    public class ClientDto
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }
}
```

### PARTE III: Creación de la Vista-Modelo

6. En el proyecto ClientUI crear una carpeta `viewmodels` y proceder a crear dentro de este los siguientes archivos y contenido:
> ViewModelBase.cs : Vista-Modelo base que implementa la interfaz INotifyPropertyChanged, que utiliza el patron Observador y permite que los controles de la interfaz se suscriban a las propiedades de las vista modelos para el control de la lógica de presentación.
```C#
using System;
using System.ComponentModel;
namespace ClientUI.viewmodels
{
    public class ViewModelBase : INotifyPropertyChanged, IDisposable
    {
        public event PropertyChangedEventHandler PropertyChanged;
        protected void OnPropertyChanged(string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
        public virtual void Dispose() { }
    }
}
```
> RelayCommand.cs : Clase que funcionara como delegada, permitiendo asociar propiedades o en esta caso comando a determinados metodos para permitir ejecutar acciones pertenecientes a la lógica de presentación
```C#
using System;
using System.Windows.Input;

namespace ClientUI.viewmodels
{
    public class RelayCommand : ICommand
    {
        private readonly Predicate<object> _canExecute;
        private readonly Action<object> _execute;

        public RelayCommand(Predicate<object> canExecute, Action<object> execute)
        {
            _canExecute = canExecute;
            _execute = execute;
        }

        public event EventHandler CanExecuteChanged
        {
            add => CommandManager.RequerySuggested += value;
            remove => CommandManager.RequerySuggested -= value;
        }

        public bool CanExecute(object parameter)
        {
            return _canExecute(parameter);
        }

        public void Execute(object parameter)
        {
            _execute(parameter);
        }
    }
}
```
> ClientViewModel.cs: Vista-Modelo que se asociara a la interfaz grafica de la ventana de mantenimiento del cliente.
```C#
using System.Collections.ObjectModel;
using ClientUI.models;

namespace ClientUI.viewmodels
{
    public class ClientViewModel : ViewModelBase
    {
        public ClientViewModel()
        {
            Clients = new ObservableCollection<ClientDto>() {
                new ClientDto() { FirstName = "Miles", LastName = "Morales" },
                new ClientDto() { FirstName = "Peter", LastName = "Parker" },
                new ClientDto() { FirstName = "Miguel", LastName = "Ojara" }
            };
        }
        
        public ObservableCollection<ClientDto> Clients {get; set;}
        private ClientDto _s_client;
        public ClientDto Client
        {
            get { return _s_client; }
            set { 
                    if (value!=null)
                    {
                        _s_client = value; 
                        OnPropertyChanged(); 
                    }
                }
        }
        
        private RelayCommand? _cmdNew;
        public RelayCommand NewCommand { 
            get {
                _cmdNew ??= new RelayCommand(
                    p => true,
                    p => NewClient());
                return _cmdNew;
            }
        }
        private void NewClient()
        {
            var _new = new ClientDto() {FirstName = "", LastName=""};
            Clients.Add(_new);
            Client = _new;
        }
        private RelayCommand? _cmdUpdate;
        public RelayCommand UpdateCommand { 
            get {
                _cmdUpdate ??= new RelayCommand(
                    p => true,
                    p => UpdateClient());
                return _cmdUpdate;
            }
        }
        private void UpdateClient()
        {
            // aqui se debe llamar al servicio de actualizacion a la BD
            System.Windows.MessageBox.Show("Client Saved", "Client Admin");
        }      

        private RelayCommand? _cmdDelete;
        public RelayCommand DeleteCommand { 
            get {
                _cmdDelete ??= new RelayCommand(
                    p => true,
                    p => DeleteClient());
                return _cmdDelete;
            }
        }
        private void DeleteClient()
        {
            Clients.Remove(Client);
        }                
    }
}
```

### PARTE IV: Creación de la Vista

7. En el proyecto ClientUI crear una carpeta `views` y proceder a crear dentro de este los siguientes archivos y contenido:
> ClientWindow.xaml
```XAML
<Window x:Class="ClientUI.views.ClientWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ClientUI.views"
        xmlns:vm="clr-namespace:ClientUI.viewmodels"
        mc:Ignorable="d"
        Title="Mantenimiento de Clientes" Width="600" Height="300">

    <Grid ShowGridLines="True">
        <Grid.Resources>
            <vm:ClientViewModel x:Key="clientViewModel"/>
        </Grid.Resources>        

        <Grid.DataContext>
            <Binding Source="{StaticResource clientViewModel}"/>
        </Grid.DataContext>

        <Grid.RowDefinitions>  
            <RowDefinition Height="2*"/>  
            <RowDefinition Height="Auto"/>  
            <RowDefinition Height="*"/>  
            <RowDefinition Height="4*"/>  
            <RowDefinition Height="2*"/>  
        </Grid.RowDefinitions>
        <Label Content="Mantenimiento de Clientes" HorizontalAlignment="Center" VerticalAlignment="Center" />  
        <Grid ShowGridLines="True" Row="1">
            <Grid.RowDefinitions>  
                <RowDefinition Height="Auto"/>  
                <RowDefinition Height="*"/>  
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>  
                <ColumnDefinition Width="Auto"/>  
                <ColumnDefinition Width="*"/>  
                <ColumnDefinition Width="Auto"/>  
                <ColumnDefinition Width="2*"/>  
            </Grid.ColumnDefinitions>
            <Label Content="Nombres" Grid.Row="0" 
                HorizontalAlignment="Left" VerticalAlignment="Top" />  
            <Label Content="Apellidos" Grid.Row="1" 
                HorizontalAlignment="Left" VerticalAlignment="Top" />  
            <TextBox Grid.Row="0" Grid.Column="1" HorizontalAlignment="Stretch" 
                Name="txtFirstName" VerticalAlignment="Stretch" Margin="5"
                Text="{Binding ElementName=UserGrid,Path=SelectedItem.FirstName}" />  
            <TextBox Grid.Row="1" Grid.Column="1" HorizontalAlignment="Stretch" 
                Name="txtLastName" VerticalAlignment="Stretch" Margin="5"
                Text="{Binding ElementName=UserGrid,Path=SelectedItem.LastName}" />  
            <Button Content="Nuevo" Grid.Row="0" Grid.Column="2" 
                HorizontalAlignment="Left" Name="btnNew"   
                VerticalAlignment="Top"
                Command="{Binding Path=NewCommand}"  />  
            <Button Content="Guardar" Grid.Row="1" Grid.Column="2" 
                HorizontalAlignment="Left" Name="btnUpdate"   
                VerticalAlignment="Top"
                Command="{Binding Path=UpdateCommand}"  />  
            <Button Content="Eliminar" Grid.Row="1" Grid.Column="3" 
                HorizontalAlignment="Left" Name="btnDelete"   
                VerticalAlignment="Top"
                Command="{Binding Path=DeleteCommand}"  />  
        </Grid>
        <ListView Name="UserGrid" Grid.Row="3" 
            ItemsSource="{Binding Clients}" SelectedItem="{Binding Client}"  >  
            <ListView.View>  
                <GridView x:Name="grdTest">  
                    <GridViewColumn Header="Nombres" 
                        DisplayMemberBinding="{Binding FirstName}" Width="80" />  
                    <GridViewColumn Header="Apellidos" 
                        DisplayMemberBinding="{Binding LastName}" Width="100" />  
                </GridView>  
            </ListView.View>  
        </ListView>  
    </Grid>
</Window>
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

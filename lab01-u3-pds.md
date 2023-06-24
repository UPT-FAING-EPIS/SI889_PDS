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

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/bdfc8fbb-e2a8-408b-83a4-b72dc7ccc82f)

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/203c2e9c-de6b-4291-9530-034fed11aca2)

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
5. Modificar el archivo App.xaml, en la propiedad StartupUri, colocar "views/ClientWindow.xaml".
```XAML
<Application x:Class="ClientUI.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:ClientUI"
             StartupUri="views/ClientWindow.xaml">
    <Application.Resources>
         
    </Application.Resources>
</Application>
```

### PARTE II: Creación del modelo

6. En VS Code, en el proyecto ClientUI crear una carpeta `models` y proceder a crear el archivo ClientDto.cs que funcionara como Modelo base para el traslado de datos e introducir el siguiente código:
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

7. En el proyecto ClientUI crear una carpeta `viewmodels` y proceder a crear dentro de este los siguientes archivos y contenido:
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

8. En el proyecto ClientUI crear una carpeta `views` y proceder a crear dentro de este los siguientes archivos y contenido:
> ClientWindow.xaml : Vista en còdigo xaml de la ventana de mantenimiento de clientes.
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
> ClientWindow.xaml.cs: clase asociada de manera nativa a los archivos de extension xaml.
```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;

namespace ClientUI.views
{
    /// <summary>
    /// Interaction logic for ClientWindow.xaml
    /// </summary>
    public partial class ClientWindow : Window
    {
        public ClientWindow()
        {
            InitializeComponent();
        }
    }
}
```
9. Ahora se requiere comprobar la aplicación para eso abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar los comandos:
```Bash
dotnet run
```
10. Si se ejecuta correctamente debería mostrar la siguiente ventana:

![image](https://github.com/UPT-FAING-EPIS/SI889_PDS/assets/10199939/cb8c8100-78cf-49e9-b1aa-d6332edabef5)

---
## Actividades Encargadas
1. Crear una nueva ventana de mantenimiento de Direcciones del cliente que se asociace a la ventana de mantenimiento del Cliente.

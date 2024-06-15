# Análisis del proceso de adición de un dominio en la aplicación Fido Universal

**autor: Abraham González Rivero**

## Flujo de la aplicación para ejecutar la operación AddDomain:

Al realizarse la acción de intentar añadir un nuevo dominio desde la vista de login, se invoca al método de *`AddDomain`* del controlador de esta vista. 

Este método se comunica con *`FidoServiceReal`*, contenedor de servicios del front,  que construye la operación a realizar, en este caso añadir un dominio, y los datos que requiere la operación, nombre del dominio y número de serie; y ,a través de *`CSharpInterface`*, envía la orden de ejecutar la operación al backend de la aplicación.

<image src="images/fidoservad.jpg" alt="FidoServiceReal.AddDomain">  

<image src="images/cintcallback.png" alt="CSharpInterface.CallCSharpCallback">

Esta orden la recibe la instancia de *`WebViewController`*(FidoUniversal\jsLayer\WebViewController.cs) contenida en el *`HybridPage`*(FidoUniversal\HybridPage.xaml.cs) del backend de la aplicación, quien deserializa el json que contiene los datos de la operación e invoca a *`callMethod`* para su ejecución.

<image src="images/wvcinit.png" alt="WebViewController.Initialize">

El método *`CallMethod`* envía el *`Message`*(representación abstracta de una operación de modo callback) al *`CoreStateManager`*(FidoUniversal\core\CoreStateManager.cs), encargado de ejecutar el tipo específico de operación, *`AddDomain`* en este caso específico.

<image src="images/wvccallm.png" alt="WebViewController.CallMethod">

El *`CoreStateManager`* para este caso utiliza reflexión para encontrar la clase encargada de realizar esta operación específica y llama al método *`execute`* quien ejecuta la lógica de la operación. Todas las operaciones heredan de la clase abstracta *`BaseOperation`* quien los obliga a implementar su propia versión de *`execute`*. De esta forma, se crea la instancia de la operación de *`AddDomain`* y se ejecuta.

<image src="images/execop.png" alt="CoreStateManager.ExecuteOperation">

## Operación AddDomain

Durante la ejecución de la operación de *`AddDomain`*(FidoUniversal\core\operation\message\AddDomain.cs), una vez mapeado los datos del dominio a añadir a su entidad correspodiente, se procede a llamar al método *`execute`* de una instancia de la clase *`AddDomainCloud`*(FidoUniversal\core\operation\synchronization\cloud\AddDomainCloud.cs).

<image src="images/addcloud.png" alt="AddDomainCloud">

Esta clase se encarga de asegurar la sincronización entre la base de datos local y la base de datos del servidor. Se comprueba que el dominio exista en la base de datos remota, si existe, se registra el dispositivo desde donde se añade el dominio y luego se actualiza la base de datos local con el nuevo dominio.

<image src="images/checkdomain.png" alt="SyncDomainCloud">

De ser exitosa esta operación se llama al método *`execute`* de la clase *`SyncAppConfigurations`*(FidoUniversal\core\operation\synchronization\cloud\SyncAppConfigurations.cs)

<image src="images/addDsapp.png" alt="AddDomain SyncAppConfigurations">

Esta clase se encarga de asegurarse de cargar 
las configuraciones de la aplicación para este dominio de la base de datos remota, y actualizar las configuraciones de la base de datos local, para luego actualizar el CoreContext con las configuraciones actualizadas.

<image src="images/synApps.png" alt="SyncAppConfigurations">

De ser exitosa esta operación se llama al método *`execute`* de la clase *`SyncMobileProviders`*(FidoUniversal\core\operation\synchronization\cloud\SyncMobileProviders.cs)

<image src="images/addDsmob.png" alt="AddDomain.SyncMobileProvider">

En esta clase se actualiza la base de datos local con los proveedores móviles cargados desde el servidor remoto.

<image src="images/smob.png" alt="SyncMobileProvider">

De ser exitosa esta operación, y si la operación no está configurada en modo presión y en modo local se llama al método *`execute`* de la clase *`StartServerConnection`*(FidoUniversal\core\operation\server\StartServerConnection.cs)

<image src="images/addDservcon.png" alt="AddDomain.StartServerConnection">

En esta clase se crea una conexión al servidor proporcionado por la configuración de la aplicación y se crea un puente entre la aplicación y el servidor.

<image src="images/servcon.png" alt="StartServerConnection">

De ser exitosa esta operación se llama al método *`execute`* de la clase *`SyncUsers`*(FidoUniversal\core\operation\server\SyncUsers.cs)

<image src="images/addDsyncuser.png" alt="AddDomain.SyncUsers">

Aquí se crea un nuevo *`DomainDatabaseManager`*() y se envian a través del puente los usuarios locales de la base de datos al servidor codificados en un txt,
con la respuesta del servidor se actualizan los usuarios de la base de datos.

El otro caso corresponde a Usuarios de presión, se escapa de los objetivos de este informe.

Finalmente se devuelve el nuevo dominio al front para ser mostrado.


# prj_ddd
- Entendiendo la arquitectura Domain Driven Design

## [ddd_cqrs-php-example-codely](https://github.com/eacevedof/prj_ddd/tree/master/ddd_cqrs-php-example-codely)
  - Esqueleto de un proyecto php en DDD

## [ddd_last-wishes-carlos-buenosvinos](https://github.com/eacevedof/prj_ddd/tree/master/ddd_last-wishes-carlos-buenosvinos)
  - App en php
  - [Playlist de Carlos](https://www.youtube.com/watch?v=uvKS6UCUZes&list=PLfgj7DYkKH3DjmXTOxIMs-5fcOgDg_Dd2)

## [ddd_online-store-yoelvis](https://github.com/eacevedof/prj_ddd/tree/master/ddd_online-store-yoelvis)
- App en c#
- Ejemplo de una webstore llevado a DDD
- **DOMAIN**
```js
├───Ordering.Domain
│	├───Common
│	│	Entity.cs (campo id)
			- modelo maestro
│	│	IAggregateRoot.cs
			- El que acotará la persistencia
│	│	IRepository.cs
			- Interface que implementa IAggregateRoot
│	│	IUnitOfWork.cs
			- clase vacia
│	│	ValueObject.cs
      - abstract class implementa IEquatable
      - tiene los métodos obligatorios para trabajar con valueobjects
│	│
│	├───Exceptions
│	│	OrderingDomainException.cs
	  - Extiende Exception. Métodos con excepciones
│	│
│	└───Model
│	├───CustomerAggregate
│	│	Address.cs (VO)
	  - valueobject con getters y setters
│	│	CreditCard.cs (VO)
	  - valueobject con getters y setters
│	│	CreditCardType.cs (VO)
	  - enum con constantes Unknown=0 a Amex=3
│	│	Customer.cs
    - extends Domain.Common.Entity implementa IAggregateRoot 
│	│	ICustomerRepository.cs
	  - interfaz implementa Domain.Common.IRepository implementa IAggregateRoot
│	│
│	├───OrderAggregate
│	│	IOrderRepository.cs
    - interfaz implementa Domain.Common.IRepository implementa IAggregateRoot
│	│	Order.cs
    - extends Domain.Common.Entity implementa IAggregateRoot
    - define el método que interactuará con OrderItem: public void AddOrderItem(int quantity, Product product)
│	│	OrderItem.cs
    - extends Domain.Common.Entity 
    - se esconde su constructor vacio para evitar que se cree una instancia inconsistente
    - define su constructor: public OrderItem(Order order, int quantity, Product product)
│	│	OrderState.cs
│	│ - tipo enumerado: Canceled=0, Pending=1, Shipped=2, Archived=3
│	├───ProductAggregate
│	│	Product.cs
    - extends Domain.Common.Entity implementa IAggregateRoot
    - esconde constructor vacio
    - ?? permite la configuración de una instancia estática vacia NullProduct ^^
│	├───Shared
│	│	Currency.cs (VO)
    - valueobject extends Domain.Common.ValueObject<Currency>
│	│	Money.cs (VO)
│	│ - valueobject extends Domain.Common.ValueObject<Money>
│	└───ShoppingCartAggregate
│	  ShoppingCart.cs
    - clase simple, no extiende de nada
    - tiene dos propiedades Buyer e Items
    - su constructor vacio esta oculto que inicializa Items
    - tiene un metodo estatico factory (CreateEmpty) que obliga a pasar un Customer 
    - tiene sus getters y setters 
    - implementa un método AddItem(int quantity, Producto product)
│		ShoppingCartItem.cs
    - el constructor por defecto está oculto e inicializa sus atributos quantity=0 y product = Product.NullProduct
    - tiene un metodo estatico factory (Create) que obliga a pasar un quantity y product 
```
- **INFRASTRUCTURE**
```js
├───Ordering.Infrastructure
│	│	OrderingContext.cs
    - extends Microsoft.EntityFrameworkCore.DbContext
    - atributo publico estático DEFAULT_SCHEMA = ordering
    - gestiona los select all de Orders, OrdersItems, Customer
    - métodos: get/set Orders, OrderItems, Customer
│	│
│	├───Mappings
│	|	CustomerMap.cs
    - implementa Microsoft.EntityFrameworkCore.IEntityTypeConfiguration
    - public void Configure(EntityTypeBuilder<Customer> customerConfiguration)
    - mapea la clase con los campos en base de datos. Se define la primarykey, requeridos, tabla etc
│	|	OrderItemMap.cs
    - implementa Microsoft.EntityFrameworkCore.IEntityTypeConfiguration 
    - public void Configure(EntityTypeBuilder<OrderItem> orderItemConfiguration)
    - mapea atributos con el esquema de datos
│	|	OrderMap.cs
    - implementa Microsoft.EntityFrameworkCore.IEntityTypeConfiguration 
    - public void Configure(EntityTypeBuilder<Order> orderConfiguration)
    - usa los métodos del ORM para la configuración de los atributos
│	│
│	└───Repositories
|   CustomerRepository.cs
    - implementa Ordering.Domain.CustomerAggregate.ICustomerRepository (Domain.Common.IRepository implementa IAggregateRoot)
    - define los métodos Add, Update, Find, FindById
|		OrderRepository.cs
    - implementa Ordering.Domain.OrderAggregate.IOrderRepository (Domain.Common.IRepository implementa IAggregateRoot)
    - constructor obliga a pasar OrderingContext
    - define metodos: Add, Get, Update
```
- **APPLICATION**
```JS
├───Ordering.Application
│   │ IOrdersService.cs
      - define un método: OrderPlacedViewModel PlaceOrder(ShoppingCartInputModel shoppingCartInputModel);
│   │ OrdersService.cs
      - implementa IOrdersService
      - tiene atributos: _orderRepository, _shippingService
      - usa: Ordering.Domain.OrderAggregate.IOrderRepository y IShippingService
      - usa: ShoppingCartInputModel  
│   │
│   ├───ExternalServices
│   │ IShippingService.cs
      - no implementa nada
      - OrderShippingConfirmation SendRequestForDelivery(Order order);
│   │ OrderShippingConfirmation.cs
      - clase básica con atributos:TrackingId, ExpectedShipDate con sus getters y setters
│   │
│   ├───InputModels
│   │ ShoppingCartInputModel.cs
│   │ - clase básica con atributo: EnableEditOnShoppingCart con sus getters y setters
│   └───ViewModel
│     OrderProcessedViewModel.cs
      - clase básica con atributos: OrderId, ShippingDetails con sus getters y setters 
      - usa OrderShippingConfirmation
```
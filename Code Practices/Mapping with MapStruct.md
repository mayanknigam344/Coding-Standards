**Mapping Rules**

**Mapstruct CONFIG parameter -**
Mapper interface annotation: @Mapper should be configured with the config parameter. It is recommended to use a separate file for it (common for similar mappers).

Note: The option **componentModel = "spring"** together with the **injectionStrategy = InjectionStrategy.CONSTRUCTOR** should be used, so the configuration always fails fast if not updated. 
If using mappers without Spring, initializing  mappers' object using constructors is needed. 
Also, **collectionMappingStrategy = CollectionMappingStrategy.ADDER_PREFERRED** should always be used. This allows for automatic generation one-to-many, many-to-one and many-to-many mappers (less code).

```
@MapperConfig(
    componentModel = "spring",
    injectionStrategy = InjectionStrategy.CONSTRUCTOR,
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS,
    unmappedTargetPolicy = ReportingPolicy.IGNORE,
    collectionMappingStrategy = CollectionMappingStrategy.ADDER_PREFERRED)
public interface OrderCreateMapperConfig {}
```

**Mapstruct USES parameter -**
Mapper interface annotation: **@Mapper** should list all the external mappers for complex types with the **uses** parameter.
Note: Only those external mappers that are actually used will be included in the implementation's constructor (listing it alone does not guarantee that).
```
@Mapper(
    config = OrderSalesInformationNotifMapperConfig.class,
    uses = {
      PaxTypeMapper.class,
      SupplierOrderTypeMapper.class})
```

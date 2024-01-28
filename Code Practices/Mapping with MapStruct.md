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
**Autowiring and using nested mappers as beans -**
On rare occasions there might be a need to autowire and use nested mappers as beans. In such cases the autowired mappers should be excluded from the usage block (this will result in using an abstract class for the mapper definition instead of an interface). Also, use a setter bean injection, otherwise instantiating the mapper without Spring will not be possible (i.e. for unit tests or for using iata-mapping library in a non-Spring project).
```
@Mapper(config = OrderCreateMapperConfig.class,
        uses = { PersonMapper.class})
public abstract class PassengerMapper {
 
  protected IdentityDocumentMapper identityDocumentMapper;
 
  @BeanMapping(ignoreByDefault = true)
  ..........
  @Mapping(target = "citizenshipCountryCode", source = "paxType.citizenshipCountryCode")
  @Mapping(target = "residenceCountryCode", source = "paxType.residenceCountryCode")
  public abstract Passenger map(PaxType paxType, DataListsType dataListsType);
 
  @AfterMapping
  protected Passenger doAfterMapping(
      @MappingTarget Passenger.Builder passenger, PaxType paxType, DataListsType dataListsType) {
    return passenger
        .addAllIdentityDocuments(mapIdentityDocumentFromIdentityDocType(paxType.getIdentityDoc()))
        .build();
  } }
 
  private List<IdentityDocument> mapIdentityDocumentFromIdentityDocType(
      List<IdentityDocType> identityDocTypes) {
    MutableInt counter = new MutableInt(1);
    return identityDocTypes.stream()
        .map(identityDocType -> identityDocumentMapper.map(identityDocType, counter))
        .toList();
  }
 
  @Autowired
  public void setIdentityDocumentMapper(IdentityDocumentMapper identityDocumentMapper) {
    this.identityDocumentMapper = identityDocumentMapper;
  }
}
```

**Using nested mappers both in USES and in @AfterMapping -**
Also on rare occasions it might be necessary for nested mappers to be both in the constructor (listed in the uses parameter) and in the @AfterMapping method.
In such cases this specific nested mapper should not be autowired but created as a regular Java object with the keyword new.
This combination, however, should be avoided if not absolutely necessary.
```
@Mapper(
    config = OrderSalesInformationNotifMapperConfig.class,
    uses = { XMLGregorianCalendarMapper.class})
public abstract class PaxTypeMapper {
 
  private XMLGregorianCalendarMapper xmlGregorianCalendarMapper = new XMLGregorianCalendarMapperImpl();
 
  @AfterMapping
  protected void mapSurname(Passenger passenger, @MappingTarget PaxType paxType) {
    paxType.getIdentityDoc().forEach(i -> setIdentityDocData(i, passenger));
  }
 
  private void setIdentityDocData(IdentityDocType identityDocType, Passenger passenger) {
    ..........
     setDateBirth(identityDocType, xmlGregorianCalendarMapper)
  }
 
  private Consumer<Date> setDateBirth(
      IdentityDocType identityDocType, XMLGregorianCalendarMapper xmlGregorianCalendarMapper) {
    return date -> identityDocType.setBirthdate(xmlGregorianCalendarMapper.mapDate(date));
  }
}
```
**Naming convention for mapping methods -**
The name of the main public mapping method(s) should be map. Helper methods can have different names and should be set as private or protected if private is not possible.

Note: Java allows to define multiple methods with the same name within one class, as long as the list of parameter types is different.

**Annotation @BeanMapping (ignoreByDefault = true) -**
Annotation @BeanMapping (ignoreByDefault = true) should be used on mapping methods. 

Note: Although automatic mapping may be a good idea in certain cases (less code, automatic mapping for the fields with the same type and name), it is also much more prone to errors than the explicitly stated mapping. Also, explicit field mapping will help to ensure that nothing new is mapped without extending tests.

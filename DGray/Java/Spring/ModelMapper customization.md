---
title: ModelMapper customization
updated: 2023-06-30 14:21:06Z
created: 2023-06-23 11:34:30Z
latitude: 59.93428020
longitude: 30.33509860
altitude: 0.0000
---

```java
@Autowired
public ContractCommentsMapper(ModelMapper modelMapper) {
    modelMapper.addMappings(new PropertyMap<ContractComment, ContractCommentDto>() {
        final Converter<ContractComment, UUID> uuidConverter =
                c -> c.getSource().getId();
 final Condition<ContractComment, UUID> notNull =
                c -> c.getSource() != null;
  @Override
  protected void configure() {
            when(notNull).using(uuidConverter)
                    .map(source.getId(), destination.getId());
  }
    });
}
```

```java
@Autowired
public EmployeeBscsMapper(BscsService bscsService, ModelMapper modelMapper) {
    EmployeeBscsMapper.bscsService = bscsService;
  modelMapper.addMappings(new PropertyMap<ContractDetailsIns, ContractDetailsInsDto>() {
        final Converter<String, UserDto> userDtoConverter =
                c -> getUserDto(c.getSource());
 final Condition<String, UserDto> notNull =
                c -> c.getSource() != null;
  @Override
  protected void configure() {
            when(notNull).using(userDtoConverter)
            .map(source.getResponsibleUser(), destination.getResponsibleUser());
            when(notNull).using(userDtoConverter)
            .map(source.getVerifyUser(), destination.getVerifyUser());
		});
}
```
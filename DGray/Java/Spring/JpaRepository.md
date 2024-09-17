---
title: JpaRepository
updated: 2021-02-27 19:41:17Z
created: 2021-02-27 19:31:19Z
tags:
  - jparepository
  - springdata
---

## JpaRepository

```java
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
  List<T> findAll();
  List<T> findAll(Sort var1);
  List<T> findAllById(Iterable<ID> var1);
  <S extends T> List<S> saveAll(Iterable<S> var1);
  void flush();
  <S extends T> S saveAndFlush(S var1);
  void deleteInBatch(Iterable<T> var1);
  void deleteAllInBatch();
  T getOne(ID var1);
  <S extends T> List<S> findAll(Example<S> var1);
  <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}

```

[[jparepository]] [[springdata]]

|     |     |     |
| --- | --- | --- |
| **Ключевое слово** | **Пример имени метода** | **JPQL код** |
| And | findByLastnameAndFirstname | … where x.lastname = ?1 and x.firstname = ?2 |
| Or  | findByLastnameOrFirstname | … where x.lastname = ?1 or x.firstname = ?2 |
| Is, Equals | findByFirstname, <br>findByFirstnameIs, <br>findByFirstnameEquals | … where x.firstname = ?1 |
| Between | findByStartDateBetween | … where x.startDate between ?1 and ?2 |
| LessThan | findByAgeLessThan | … where x.age < ?1 |
| LessThanEqual | findByAgeLessThanEqual | … where x.age <= ?1 |
| GreaterThan | findByAgeGreaterThan | … where x.age > ?1 |
| GreaterThanEqual | findByAgeGreaterThanEqual | … where x.age >= ?1 |
| After | findByStartDateAfter | … where x.startDate > ?1 |
| Before | findByStartDateBefore | … where x.startDate < ?1 |
| IsNull | findByAgeIsNull | … where x.age is null |
| IsNotNull,NotNull | findByAge(Is)NotNull | … where x.age not null |
| Like | findByFirstnameLike | … where x.firstname like ?1 |
| NotLike | findByFirstnameNotLike | … where x.firstname not like ?1 |
| StartingWith | findByFirstnameStartingWith | … where x.firstname like ?1(parameter bound with appended %) |
| EndingWith | findByFirstnameEndingWith | … where x.firstname like ?1(parameter bound with prepended %) |
| Containing | findByFirstnameContaining | … where x.firstname like ?1(parameter bound wrapped in %) |
| OrderBy | findByAgeOrderByLastnameDesc | … where x.age = ?1 order by x.lastname desc |
| Not | findByLastnameNot | … where x.lastname <> ?1 |
| In  | findByAgeIn(Collection&lt;Age&gt; ages) | … where x.age in ?1 |
| NotIn | findByAgeNotIn(Collection&lt;Age&gt; ages) | … where x.age not in ?1 |
| True | findByActiveTrue() | … where x.active = true |
| False | findByActiveFalse() | … where x.active = false |
| IgnoreCase | findByFirstnameIgnoreCase | … where UPPER(x.firstame) = UPPER(?1) |
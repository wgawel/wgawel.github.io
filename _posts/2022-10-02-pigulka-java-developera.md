---
layout: post
title: "[PL] Źródła wiedzy Java Developera"
categories: [Java]
---

> Artykuł aktualizowany (ostatnio 9.11.2022)

Materiałów sieci jest mnóstwo, tutaj wybrałem te, które osobiście uważam najbardziej merytoryczne i przydatne.
Do części załączam własne notaki i streszczenia.

## Table of contents
1. [Porty i adaptery aka Hexagonal architecture, CQRS](#hexagonal)
   * [Jakub Nabrdalik: Keep IT clean, or how to hide your shit](#keep_it_clean)
   * [(Udemy) Mateusz Chrzonstowski: Architektura aplikacji](#chrzonstowski_architektura)
2. [JPA/Hibernate](#jpa_hibernate)
   * [Jakub Kubryński: JPA - beyond copy-paste](#jpa_beyond)
3. [Architektura - Model C4](#architektura_c4)
   * [Jakub Kubryński: Architektura i architekt AD2022](#kubrynski_architektura)
4. [Spring](#spring)
   * [Jakub Nabrdalik: Jinkubator #9 - Spring Framework](#jinkubator_spring)
   * [Jakub Kubryński: Jinkubator #25 - Spring Boot](#jinkubator_spring_boot)
5. [Testowanie, TDD](#testowanie)
    * [Jakub Nabrdalik: Test Driven Traps](#nabrdalik_test)
6. [Wydajność](#wydajnosc)
    * [Marcon Skalski: Wszystko co chcesz wiedzieć o wątkach, ale boisz się zapytać.](#skalski_watki)
7. [Inne bazy wiedzy](#inne_bazy_wiedzy)
    * [Materiały Bottega](#bottega)
    * [Krzysztof Góralski: baza wiedzy](#kgoralski)

---

## Porty i adaptery aka Hexagonal architecture<a name="hexagonal"></a>

### Jakub Nabrdalik: Keep IT clean, or how to hide your shit<a name="keep_it_clean"></a>
Jakub porusza tutaj także temat CQRS i dobrych praktyk przy implementowaniu nowych wymagań projektu.

<iframe width="560" height="315" src="https://www.youtube.com/embed/KO6K3_Ac54M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<details>
    <summary>Moje notatki z prezentacji</summary>
<pre>
Usunąć public z templateów klas w InteliJ.
Public tylko bardzo uzasadnione (dla fasad).

Plusy:
- można dowolnie modyfikowac implementację, API do building blocku jest w fasadzie.
- komunikujemy się przez DTO, możemy wyrzucać wyjątki - one też powinny byc publiczne, struktura:
  |-article (package)
  |-domain (package)
  |-dto (package)
  |-exceptions (package)
  |-Article.java
  |-ArticleFacade (public - only this one)
  |-ArticleFactory
  |-ArticlePreviewer
  |-ArticlePublisher
  |-ArticleRejector
  |-ArticleRemover
  |-ArticleRepository (interface)
  |-ArticleRules
  |-ArticleStatus (enum)
  |-ArticleSubmitter
  |-ArticleUnpublisher
  |-ArticleUpdater
  |-Category
  |-CategoryPath
  |-StatusDependentOperation

Kotlin nie obsługuje package scope, Groovy przez adnotację, Scala może dodać ten scope do protected i private.

Używaj CQRS

https://dzone.com/articles/the-java-8-api-design-principles

Używaj klas wewnętrznych (w klasach):
Wyjątki krytyczne (których nikt nie przechwyci, ja: np. w testach).
Klasy używane tylko przez tą klasę.

Nie dziedzicz po klasach, (tylko po interfejsach).

Baza danych w pamięci poprzez ConcurrentCollect (uzupełnić)

Nie korzystaj z Autowired (bo tworzy publiczne klasy, dostępne wszędzie, brak modułowości). Zamiast tego sam twórz konfiguracje.

---

Jak robić projekt:
1. Wymagania
2. Wypisać testy akceptacyjne. (Implementuje go, jest pusty).
3. Implementuję domene, dzielę na jakieś fragmenty. Zazwyczaj kilka bounded-contextów, czy innych większych klocków.
4. Implementuję dany fragment zaczynając od fasady.
5. Na początku robię bazę danych w pamięci (wyżej).
6. To wszystko powinno śmigać w UnitTestach, cała logika powinna działać.
7. Wystawiam to przez Controler. Bardzo prosty.
8. Dodaję bazę danych (np. MongoDB w implementacji przez SpringData).
9. Sprawdzam, czy to działa w teście akceptacyjnym.

Efekt: 1-5 testów akceptacyjnych, 60 testów jednostkowych działających na fasadzie. Wykonywanie: 1s.


Przykład:
1)
Wymagania zdefiniowane razem z klientem. Powstając testy akceptacyjne, czyli scenariusz testowy. Przepisanie storyjki na testy akcpetacyjne i unitowe.
Stuff client asked for:

    CreateArticleAcceptanceSpec
    DeleteArticleAcceptanceSpec
    PreviewArticleAcceptanceSpec
    PublishArticleSuccessAcceptanceSpec
    RejectArticleAcceptanceSpec
    UnpublishArticleAcceptanceSpec
    UpdateArticleAcceptanceSpec

Stuff UX asked for (client took for granted)

    FetchArticleListByAuthorAcceptanceSpec
    FetchArticleListByTitleAcceptanceSpec
    SearchArticleByAuthorAcceptanceSpec
    SearchArticleByBioAcceptanceSpec
    SearchArticleByIdAcceptanceSpec
    SearchArticleByStatusAcceptanceSpec
    SearchArticleByTitleAcceptanceSpec
    SearchArticleByTitleAndQueryAcceptanceSpec

2) Co potrzebuję (szkic):

title keep IT clean example

User->Controller: create article
Controller->Controller: http logic
Controller->Facade: CreateArticle
Facade->Facade: validate
Facade->Factory: create
Factory-->Facade: DraftArticle
Facade->Repository: save
Facade-->Controller: ArticleQueryDto
Controller-->User: json

Klasy:
- Controller
- Repository
- Factory
- Article
- Configuration (@Configuration with @Beans)
- Facade
  Żadna klasa nie musi być pubiczna. (Tylko JUnit wymaga).
  Which should be public: API for your domain (bounded context or aggregate root), that other can use.
  Publiczna powinna być tylko Facade.
  Public data structures (nested in Facade for now).
- CreateArticleDto
- ArticleQueryDto

Controllers do not have to know anything except for Facade & DTOs, so we can move them out.

DDD (Domain Driven Design) - dzielimy aplikację na fragmenty. opowiada o "Bounded Contextach". Ale też "Agretach Rootach".
Artykuł to Bounded Context.
Pozwala też korzystać z SSH w SpringBoot. Pozwala np. wykonać komendę importu artykułów z jakiegoś źródła. <- to jest część publiczna.

3) Struktura plików: (konwencja nazw allegro: Endpoint, czyli Controller; wszystko package scope poza Facade)
   article/
   ├── ArticleActionsEndpoint.java
   ├── ArticleContentValidationEndpoint.java
   ├── ArticleCrudEndpoint.java
   ├── UpdateArticleDtoVersusPathParameterValidator.java
   ├── domain
   │   ├── Article.java
   │   ├── ArticleConfiguration.java
   │   ├+─ ArticleFacade.java
   │   ├── ArticleFactory.java
   │   ├── ArticlePreviewer.java
   │   ├── ArticlePublisher.java
   │   ├── ArticleRejector.java
   │   ├── ArticleRemover.java
   │   ├── ArticleRepository.java
   │   ├── ArticleRules.java
   │   ├── ArticleStatus.java
   │   ├── ArticleSubmitter.java
   │   ├── ArticleUnpublisher.java
   │   ├── ArticleUpdater.java
   │   ├── Category.java
   │   ├── CategoryPath.java
   │   ├── StatusDependentOperation.java
   │   ├── dto/...
   │   └── exceptions/...

Plus: Wiemy dokładnie co z czego korzysta.

4) Wyciąganie danych na wierzch (query/search) (czyli 90% aplikacji, reszta to logika biznesowa)
   Dla GUI:
- FetchArticleListByAuthorAcceptanceSpec
- FetchArticleListByTitleAcceptanceSpec
- SearchArticleByAuthorAcceptanceSpec
- SearchArticleByBioAcceptanceSpec
- SearchArticleByIdAcceptanceSpec
- SearchArticleByStatusAcceptanceSpec
- SearchArticleByTitleAcceptanceSpec
- SearchArticleByTitleAndQueryAcceptanceSpec

Wzorzec CQRS (Command Query Responsibility Segregation).
cqrs1.png
Jeśli wykonujemy akcje na domenie, które zmieniają stan (działanie aplikacji, sens) to:
Command:
|          application                                                                                                       | Infrastructure
GUI -> CommandFacade:doSth(Command command) -> AggregateRoot(entity1):property1/method1() (kompozycja z Entity2, Entity3)  -> InfrastructureService -> DB

Fasada odpala encje.

Query (bezpośrednio z widoku do bazy):
GUI -> QueryFacade:getSth(Query query) -> DB

5)
@Document(collection = MongoCollections.ARTICLES) // jaka kolekcja
@Type("articles") // dla javersa do wersjonowania
class Article implements DraftArticle, SubmittedArticle, DeletedArticle, RejectedArticle, PublishedArticle (...)

Implementuje tyle interfesjów, bo zarządzam przy pomocy tych interfejsów cyklem życia Artykuły. Ma maszynę stanową - może znajdować się w różnych stanach.

--

@Getter
@Builder
@Document(collection = MongoCollection.ARTICLES)
@ToString(exclude = "content")
@AllArgsConstructor
public class ArticleQueryDto {
//Mapping only data I want to share
}

ArticleQueryDto - rzecz, którą wyciągam z bazy dancyh bezpośrednio na frontend. Bezpośrednie zapytanie do db. Pole wspólne z Article, ale tylko te które chcę wyciągać na frontend (ograniczony subset w porównaniu do Article). Zgadza się mniej więcej z bazą danych.
O ile Article nie ma żadnych getterów, setterów, ma tylko metody które mają sensowane zachowanie, a tworzy się go przy pomocy innego obiektu np. creatora, czy fabryki. ArticleQueryDto ma gettery, buildery, constructor. Ale to jest Document MongoDB oparty o tę sama kolekcję.

article/
├── ArticleQueryEndpoint.java
└── query
├+─ ArticleQueryDto.java
├+─ ArticleQueryRepository.java // tak jak Facade
└+─ ArticleSearchParams.java

--
Następny pakiet (autorzy artykułów):

|-bio
|-domain
|-dto
|-Bio  [class]
|-BioConfiguration [class]
|-BioFacade [public class]
|-BioFactory [class]
|-BioRepository [interface]
|-query
|-BioQueryDto [public class]
|-BioQueryRepository [public interface]
|-BioEndpoint [class]

6) infrastructure (np. połączenie z innym mikroserwisem)

W DDD jeżeli mamy coś z infrastruktury, z poza domeny, to mamy tyko tego interfejs. Nic nie wiemy o implementacji.

public interface CategoryTreeClient

On może znajdować się w pakiecie "Article" albo "SharedKernel".
Potem mam jego implementację w zupełnie innym pakiecie:

|-infrastructure
|-category
|-CategoryClientCacheConfiguration [class]
|-CategoryClientConfiguration [class]
|-CategoryDtoWrapper.java
|-CategoryContainer [interface]
|-CategoryDroWrapper [class]
|-CategoryListWrapper [class]
|-ParentIdWrapper [class]
|-CategoryTreeRestTemplateClient [class]

Żadna klasa nie jest publiczna. Tylko przez interfejs, którą implementuje jedna z nich.

Gdybyście korzystali jeszcze z adnotacji @Autowired to wystarczy, ale nie róbcie tego.
Dodaj konfigurację:

@Configuration
class CategoryClientConfiguration {
@Beans
CategoryTreeClient categoryTreeClient(...) {
...
return new CategoryTreeRestTemplateClient(...);
}
}

Jak robic konfigurację:
@Configuration
class ArticleConfiguration {
@Bean
ArticleFacade articleFacade(ArticleRepository articleRepository, ReaderClient readerClient, ArticleFactory factory, FrontendProperties frontendProperties) {
ArticleUpdater updater = new ArticleUpdater(articleRepository);
ArticleSubmitter submitter = new ArticleSubmitter(articleRepository);
ArticleRemover remover = new ArticleRemover(articleRepository);
ArticleRejector rejector = new ArticleRejector(articleRepository);
ArticleUnpublisher unpublisher = new ArticleUnpublisher(articleRepository, readerClient);
ArticlePublisher publisher = new ArticlePublisher(articleRepository, readerClient);
ArticlePreviewer previewer = new ArticlePreviewer(readerClient, frontendProperties, articleRepository);
return new ArticleFacade(updater, submitter, remover, rejector, unpublisher, publisher, factory, previewer, articleRepository);
}
--
Hexagonal architecture
--
Testowanie package scope:
(...)
--

BDD (Behavior Driven Development) - testujemy zachowanie fragmentów aplikacji (nie Unitów czy klas). Fragment to pakiety z publicznym interfejsem. Nie testuje niczego w środku, tylko fasadę. Żadna zmiana w środku pakietu nie narusza kontraktu - nie trzeba poprawiać testów.
Prawdziwe BDD.
Pakiety do testów:
|-pl.allegro.content.popart.editor
|-article
|-base
|-domain
|-list
|-search
|-security

Production code:
|-pl.allegro.content.popart.editor
|-article
|-domain
|-query
|-ArticleActionsEndpoint
|-ArticleContentValidationEndpoint
|-ArticleCrudEndpoint
|-ArticleQueryEndpoint
|-UpdateArticleDtoVersusPathParameterValidator [interface]

Pakiety testów nie odbijają jeden do jednego pakietów w produkcji.
Guava @VisibleForTesting - tutaj nie potrzebne.

James Gosling, twórca Java: Gdyby jeszcze raz mógł tworzyc Javę: zrezygnowałby z dziedziczenia po klasach, zostawiłby tylko dziedziczenie po interfejsach. Używałby tylko delegacje.
Tylko w Spocku jest to potrzebne.

Antywzorce dot. dziedziczenia:
1) incoming and outgoing invoice
   Jedna klasa faktur, jedna tabela bazy danych. Jaki błąd? Use case'y (przypadki użycia) faktury sprzedażowej i zakupowej są zupełnie inne. Na poziomie domenowym nie powinno się tego modelować jako jednej faktury. 'Przypadkowo' te fakury mają te same pola. De facto, nie ma tu wspólnego interfejsy (poza drukowaniem).
   Nie dziedziczymy dlatego że mamy te same dane!
   Bierzemy pod uwagę zachowania, a nie stan. Na tym polega program. obiektowe.
2) square extending rectangle problem - coś jest czymś
   kwadrat jest specyficznym przypadkiem prostokąta? tak, ale nie jest wykonalne dziedziczenie. Naruszamy "Liskov substitution principle". Tam gdzie mamy prostokąty powinniśmy móc wstawić kwadrat. Tymczasem prosotkąt ma zachowanie: jeżeli zmienisz jedną z jego długości to druga pozstanie niezmieniona. Zachowanie kwadrata jest inne.
   Powinniśmy patrzeć na wspólne zachowania. (czyli implementacje interfejsów).
3) Mamy takie same dane, te same use casey, trochę inne zachowania?
   Builder, który buduje boxa. W zależności który box chcę uzyskać, tworze ten box przekazując mu zachowanie, którę chcę (lambda).
   Jak dostajemy konkretne implementacje, jakie są nam potrzebne? Nie tak, że za każdym razem składamy wszystko, tylko builder:
   public class OfferBoxBuilder implements BoxBuilder {
   public static OfferBoxBuilder offerTiles() {
   return new OfferBoxBuilder("[box:offerTiles]",
   (frameBuilder, offerBox) ->
   frameBuilder.offerTilesBox(offerBox),
   (frameBuilder, multiOfferBox) ->
   frameBuilder.offerTilesMultiBox(multiOfferBox));
   }
   public static OfferBoxBuilder offerCarousel() {
   return new OfferBoxBuilder("[box:offerCarousel]",
   (frameBuilder, offerBox) ->
   frameBuilder.offerCarouselBox(offerBox),
   (frameBuilder, multiOfferBox) ->
   frameBuilder.offerCarouselMultiBox(multiOfferBox));
   }
   ...
   }
   OfferBox tilesBox = OfferBoxBuilder.offerTiles()
   OfferBox carouselBox = OfferBoxBuilder.offerCarousel()

Podsumowanie:
1) Domyślny widoczność package scope ma duży sens. Jeżeli ktoś używa słówka "public" musisz mieć wytłumaczenie.
   Wprowadzi do hexagonal architecture, dobrą bazę do UnitTestów.
   Dostajecie "loose coupling" (luźne powiązanie) i "high cohesion" (wysoka spójność, czyli dobrze zdefiniowane, co robi klasa, nie robi rzeczy ze sobą niepowiązanych (np. obliczenie i wyśiwetlanie okna)).
   Klasy wewnętrzne - są dobre. Jako static wyjątki, klasy używane tylko wewnątrz.
   Hexagonal architecture / Ports & Adapters
   Jest to prostsze niż dodawanie słówka public. Użyj CQRS kiedy potrzebujesz czegoś na zewnątrz.
</pre>
</details>

### Mateusz Chrzonstowski: Architektura aplikacji<a name="chrzonstowski_architektura"></a>

Jest też dobry kurs na Udemy (płatny), jeśli ktoś preferuję taką formę przyswajania wiedzy:

[Mateusz Chrzonstowski: Architektura aplikacji](https://www.udemy.com/course/architektura-java/)

---

## JPA/Hibernate<a name="jpa_hibernate"></a>

### Jakub Kubryński: JPA - beyond copy-paste<a name="jpa_beyond"></a>

<iframe width="560" height="315" src="https://www.youtube.com/embed/UPWkpl5PL_w" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<details>
    <summary>Moje notatki z prezentacji</summary>
<pre>
2016 - Jakub Kubryński - JPA - beyond copy-paste

@Entity
public class Product {

	@Id
	@GeneratedValue
	private Long id;
	
	@Column // it does exactly nothing
	private String name;
	
	// ...
}

Hibernate automatycznie perzystuje wszystkie pola encji.
Adnotacji @Column używamy gdy chcemy np. dodać nullable albo podać nazwę kolumny.

---

@Transactional
@Service
public class ProductService {

	public void updatePrice(Long productId, Moeny newPrice) {
		Product product = productRepository.find(productId);
		product.setPrice(newPrice);
		productRepository.save(product); // id does exactly nothing
	}
}

Mechanizm dirty-checking: śledzi i sprawdza stan encji.
Przy wyjściu z z tej tranzakcji/metody zapisze ją.

---

Adnotacje na getterach i polach są podobne. Ale musi być to jednolite.
Wystarczy jedna adnotacja na get'erze zmienia cały dostęp do encji. Mixed access musi być wymuszony.
Jeśli nie potrzenujemy get'eró to adnotacje na polach.

---

Zmiana nazwy tabeli.
Domyślnie na podstawie nazwy encji.

@Entity(name = "orders")
public  class Order {
}

Powyższe zmienia nazwę encji, wiec poniższe zapyteni hql'owe przestanie działać:

em.createQuery("SELECT o FROM orders o").

Prawidłowa zmiana nazwy tabeli:

@Entity
@Table(name = "orders")
public  class Order {
}

---

MAPOWANIA

OneToOne

@Entity
public class Customer {

	@OneToOne
	private Address address;
}

@Entity
public class Address {

	@OneToOne(mappedBy = "address")
	private Customer customer;
}

mappedBy wskazuje na stronę po której będzie klucz obcy.
Bez tego będzie w obydwu tabelach.

Moja uwaga: mappedBy wskazuje na pole w przeciwnej encji. Informuje Hibernate by nie mapował w bieżącej encji.
W tym przypadku w tabeli "Customer" zostanie utworzone pole "address_id".

---

OneToMany

@Entity
public class Customer {

	@OneToMany
	@JoinColumn(name = "cutomer_id")
	private Set&lt;Address&gt; addresses;
}

@JoinColumn -> wymagane, bo domyślnie Hibernate utworzy niepotrzebną tabelę pośredniczącą.

--

ManyToMany

@Entity
public class Customer {

	@ManyToMany
	private Collection&lt;Address&gt; addresses;
}

@Entity
public class Address {

	@ManyToMany
	private Collection&lt;Customer> customers;
}

Domyślnie Hibernate tworzy dwie tabele pośredniczące.

Warto sprawdzać logi Hibernate.

---

Lazy Loading

LazyLoading w Hibernate:
- nie zawsze proxy CGlib
- jeżeli są kolekcje, to daje tam swoją implementację - to 90% przypadków, dlatego większość encji może być "final".

--

@OneToMany
Set&lt;Product> products;

Hibernate podstawia swoją implementację Set.
Wydajne - stosować Set gdzie tylko się da.

--

@OneToMany
List&lt;Product> products;

Hibernate tworzy Bag - torbę.
PersistedBag - nie pozwala na duplikaty i nie ma kolejności (sic!)


Jeżeli chcemy kolejność (prawdziwą listę) dodajemy @OrderColumn

@OneToMany
@OrderColumn
List&lt;Product> products;

Niewydajne w przypadku tabel łączących.

---

Problem N+1 select

@Entity
public class User {

	@OneToMany
	private List&lt;Address> addresses;

}

List&lt;User> users = em.createQuery("SELECT u FROM User u").getResultList();

for (User user : users) {
for (Address address " user.getAddress()) {
...
}
}

Z powodu LazyLoading Hibernate wykona liczbaUzytkowników+1 zapytań.

Jak wykryć problem N+1 ?
- patrząc na logi
- tworzać test, który pobierze obiekt statystyk Hibernate i sprawdzi ilość zapytań.

---

Jak można to poprawić?

1) @BatchSize(size = 10)

@Entity
public class User {

	@OneToMany
	@BatchSize(size = 10)
	private List&lt;Address> addresses;	
}

kiepskie rozwiązanie - mamy teraż problem (N+1)/10 zapytań.

--

2) Wykorzystać JOINY

SELECT DISTINCT u FROM User u
JOIN FETCH u.addresses

(Można też użyć Criteria API)

Jeżeli mamy Bag (torby) to możemy tylko raz łączyć tak tabele.
Hibernate wyrzuci wyjątek.

---

HOW TO SAVE

EntityManager.persist()
vs
EntityManager.merge()

Persist - działa dla nowych encji.
Jest typu void. Po wyjściu z metody przekazan encja jest encją zarządzaną.

Merge dodatkowo robi select, bo sprawdza czy encja już istnieje.
Po wyjściu zwraca encję zarządzaną, ale to co przekazaliśmy nie jest encją zarzadzaną.

Z powodów wydajnościowych zawsze stosujemy persist. Merge tylko wtedy gdy encja jest odłączona, np. zwróciła zdeserializowana z jakiejść warstwy o chcemy ją zapisać.

---

OPTIMISTIC LOCKING

@Entity
public class User {

    @Version
	private int version;
}

Gdy wersja się nie zgadza Hibernate rzuca OptimisticLockException

Problem w REST.

Powinniśmy zawsze przekazywać wersję dalej, by ją sprawdzać.

---

IDENTITY
equals() and hashcode()

Kontrakt:
- hashcode() nie może się zmienić przez cały cykl życia obiektu
- jeżeli equals zwraca true to obiekty muszą mieć ten sam hashcode.

Nie można hackcode opierać na zmieniających się polach.

Do każdej encji trzeba np. dodać UUID.

@MappedSuperclass
public abstract class BaseEntity implements Serializable {

    @Id
	@generatedValue
	private Long id;
	
	provate String uuid = UUID.randomUUID.toString();
	
	public int hashCode() {
	    return Objects.hash(uuid);
	}
	
	public boolean equals(Object that) {
	    return this == that || that instanceof BaseEntity
		            && Objects.equals(uuid, ((BaseEntity) that).uuid);
	}
}

---
CACHING

3 poziomy Cache w Hibernate
- L1 - EntityManager cache / Session cache
- L2 - EntityManagerFactory cache / SessionFactory cache
- QueryCache

Cache L1 jest zrzucany do bazy danych dopiero gdy wykonamy flush

Flush modes:
- MANUAL
- COMMIT
- AUTO (COMMIT plus gdy Hibernate wykryje zapytanie na encji, która już jest w cache L1 (zmieniła się)).
- ALWAYS (przed każdym zapytanie wykonuje flush)

LEVEL 1 CACHE PITFALLS
Gdy mamy zapytania batchowe robić flush() i zaraz clear().
Inaczej sprawdzanie cachce L1 będzie długo trwało.

--

LEVEL 2 CACHE PITFALLS
Problemy przy distributed environment - nie działa.

--

HQL injection

String hqlQuery = "SELECT p FROM Product p where p.category = '" + cat + "'";

List&lt;Product> products = em.createQuery(hqlQuery, Product.class).getResultList();


Wykorzystywać Prepared statements.
--

DYNAMIC UPDATES

@Entity
@DynamicUpdate
public class MyEntity {
/// ...
}

Hibernate domyślnie przy aktualizacji wysyła wszystkie pola encji w zapytaniu.
Jeśli tego nie chcemy używany @DynamicUpdate lub @DynamicInsert.
</pre>
</details>

---

## Architektura - Model C4<a name="architektura_c4"></a>

### Jakub Kubryński: Architektura i architekt AD2022<a name="kubrynski_architektura"></a>
Wprowadzenie to tematu współczesniej architektury aplikacji z dobrymi praktycznymi przykładami.

<iframe width="560" height="315" src="https://www.youtube.com/embed/HwvV3pjilXk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

## Spring<a name="spring"></a>

### Jakub Nabrdalik: Jinkubator #9 - Spring Framework<a name="jinkubator_spring"></a>

<iframe width="560" height="315" src="https://www.youtube.com/embed/RQEsSCwsRf0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Jakub Kubryński: Jinkubator #25 - Spring Boot<a name="jinkubator_spring_boot"></a>

<iframe width="560" height="315" src="https://www.youtube.com/embed/zQll41ha5_g" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

## Testowanie, TDD<a name="testowanie"></a>

### Jakub Nabrdalik: Test Driven Traps<a name="nabrdalik_test"></a>

<iframe width="560" height="315" src="https://www.youtube.com/embed/wbAtJlbRhbQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<details>
    <summary>Moje notatki z prezentacji</summary>
<pre>
Notatki 2013 - Jakub Nabrdalik - Test Driven Traps

Książka: Test Driven Development by Example

Dlaczego TDD?
- bezpieczeństwo
- szybki feedback, czy to działa ta jak mi się wydaje
- komunikacja, definiuje wymagania wobec aplikacji lub fragmentów kody
- najlepszy możliwy design - ewolucja tego jak wygląda nasze api, nasza aplikacja

Kiedy piszemy testy?
- PRZED pisaniem kodu (!wymagane)
- PODCZAS pisania kodu
- PO - nowe wymaganie, nowa nieznaleziona ściażka

Zasada 1: Testujemy tylko to, co mówi nazwa testu.

Zasada 2: Testujemy tylko jedną rzecz.

Zasada 3: Testuj zachowanie, nie algorytm.

Stub: On ma być/może zalogowany

Mock: Weryfikujemy, jakie interakcje zaszły.

Zasada 4: Weryfikować tylko istotne rzeczy? (czy to ma znaczenie? Czy powinienem to testować w unit testach?)

Zasada 5: Napisz kod, który ułatwi ci testowanie.

Chain<>-Merchant<>-Product
<>-Outlet<>-Terminal<>-TransactionAttempt<>-Transaction

given:
Chain chain = new Chain(corporateName:'someName', tradingName:'tradingName')

	Merchant merchant = new Merchant(name:'merchant')
	chain.addToMerchants(merchant)
	Outlet outlet = new Outlet(name:'outlet')
	merchant.addToOutlets(outlet)
	Terminal = ...

Powyższe duplikowane w każdym teście.
Najprostsze: jakie są poprawne wartości dla konstruktora:
Chain chain = new Chain(GOOD_CHAIN_PROPERTIES);
Ale to słabe: one mogą się zmienić. Więc: wzorzec fabryki. Stworzyć fabrykę i wyprodukować obiekt.
Chain chain = ChainFactory.createAndSave()

Jeżeli potrzebujemy wiele obiektów powiązanych (Unit testy i testy integracyjne):

given:
DeepChainStructure deepChainStructure = DeepChainStructure.createAndSave()

	deepChainStructure.chain
	deepChainStrucutre.terminal
	deepChainStructure.product

Zasada 6: Zbuduj domene taką jak na produkcji.

Kiedy masz bardziej skomplikowane rzeczy (np. transakcje bankowe).

class TransactionAttemptBuilder {
TransactionAttempt build(PaymentCommand command) {
...
}
}

Jeśli mam klasę, która tworzy TransactionAttempt (logikę wyciągam z konstruktora do osobnej klasy), to mogę ją wykorzysdtać także w  Unit testach tworząc Adapter, który wystawi mi takie API jakie bym chciał mieć teście. Np. potrzebuję konkretne wartości w teście. Całą resztę załatwiam przy pomocy tego samego kodu co na produkcji.

Zasada 7: Wydajność jest kluczowa dla testów.
Testy trwają powyżej 3 minut - tylko uruchamiany czasami przed commitem. Do 30 sekund - zawsze je uruchamiają.
Najlepiej podzielić testy na wolne i szybkie:
wolne: dobijąją się do bazy, do webserwera, gadają po kablu, dotykają filesystemu.
szybkie: reszta

Przykład marnowania czasu:
def "should find last dealer for smart card"() {
given:
createDealerWithTransaction(controller, "123")

	// Date must be different with seconds precision
	sleep(2000)
	Dealer secondDealer = createDealerWithTransaction(controller, "321")

when:
Dealer lastFoundDealer = transaction.findLastDealer(command.smartCardNumber)

then:
lastFoundDealer == secondDealer
}

Mniej trywialny przykład marnowania czasu:

def "findByOwner should return products with pagination"() {
given:
int numberOdProductsAtTheBeginningOfIntegrationTest = Product.count()
Chain chain = createChainAndInjectToController()
(1..11).each{ ProductFactory.createAndSaveProductFor(chain, [productCategory: ProductCategoryFactory.createAndSaveProductCategory()])}
ProductCommand productCommand = new ProductCommand([productOwnerType: ProductOwnerType.CHAIN, ownerId: chain.id])
int oldProductCount = Product.count()

when:
def list = controller.findByOwner(productCommand, chain, SearchParameters.mapWithDefaultPagination(controller.params))

then:
list.size() == 10
list.getTotalCount() == 11
Product.count() == 11 + numberOfProductsAtTheBeginningOfIntegrationTest
}

Baza danych: 3 modele:
1) (najgłupszy) bierzemy bazę z produkcji i testujemy tutaj.
2) baza danych jest pusta, test wprowadzają dane i robią rollback, czyli po testach jest pusta
3) Predefiniowane dane odbite w systemie - ustalony stan przez nas (insertami lub uruchomioną logiką biznesową).

class PrepopulatedDeepChainStructure {
static final Long chainId = 1
static final Long merchaintId = 1
static final Long outletId = 1
static final Long terminalId = 1
private Chain chain

Chain getChain() {
if(chain == null) {
chain = Chain.get(chainId)
}
return chain
}

Może same Unit testy?
Czy ma sens robienie Unit testów do DAO albo Repozytorium bez bazy danych?
Mnie interesuje czy baza zrozumiała co do niej gadam i czy wynik jest taki jaki oczekuję. Albo czy ja zrozumiałem co mi zwróciła.
Testy DAO maja sens tylko wtedy gdy jest tam jakaś logiką - ale ta jest tam bardzo rzadko.

Zasada 8: Nie mockuj wszystkiego. (fałszywe poczucie bezpieczeństwa jest gorsze od braku bezpieczeństwa).

Na przykład bez sensy: sprawdzanie czy są adnotacje JPA
Czasami warto coś przetestować w RESTcie.

UNIT
Zasada 9: Testy dla Unitów nie dla klas!
-
Zadanie przykładowe
Jasno sprecyzowane wymagania można przepisać linia po linii:
|-BookingCalendarGeneratorTest [class @Test]
|-shouldPrepareBookingCalendar():void
|-noPartOfMeetingMayFallOutsideOfficeHours():void
|-meetingsMayNotOverlap():void
|-bookingsMustBeProcessedInSubmitOrder():void
|-orderingOfBookingSubmissionShouldNotAffectOutcome():void
|-rubbishInputDataShouldEndWithException():void
|-emptyInputDataShouldEndWithException():void
|-bookingCalendarGenerator:BookingCalendarGenerator = new BookingCalendarGenerator()

1. Zapisujemy je w postaci pustych testów i zaczyamy kodować.
2. Zaczynamy kodować i przez to, że mamy fajne wzorce, wiemy co to jest extract method, wiemy co to kompozycja - wyciągamy fragmenty logiki. I wiemy, że to należy do np. tego obiektu. Bo podstawowy obiekt, który testuję, robi za wiele rzeczy. Ale on nie jest publicznym obiektem, jest mój własny.

|-eu.solidcraft [package]
|-Booking [private class]
|-BookingCalendarGenerator [public class] <- jedna klasa publiczna
|-CalendarPrinter [private class]
|-LinesGetter [private class]
|-OfficeHours [private class]
|-OfficeWorkHoursReader [private class]

Są w relacji kompozycji. Mam nadal jeden plik z testem.
 [class] BookingCalendarGenerator <>- LinesGetter
 <--<< create >>--/

Mam pokrycie 100% metod, 95% linii kody (bez jednego getera).

Zasada 10: Zawsze z góry na dół. Od wymagania, do wszystkich szczegółowych przpadków, do dostępu do bazy.

Czy nie można zrobić tylko testów integracyjnych?
Nie bo, trwają dłużej niż 30 sekund.
Bardzo cieżko jest poukładać sobie wymagania do naszych testów, do naszego kodu w testach integracyjnych.
Np. "Jeżeli w trakcie przetwarzania tranzakcji zdechnie połączenie do bazy danych, ale nie przy pierwszym insercie, ale przy drugim updacie..." <- to nie jest łatwo zrobić w teście integracyjnym. W testach jednostkowych bez problemu: bo wiem że driver do bazy danych rzuca odpowiedni exception. Zmockuję sobię driver do bazy danych, rzucę odpowiedni exception i będe wiedział że moje zachowanie jest mniej więcej ok.

Zasada 11: Testy są normalnym OO code. Nie rób tutaj rzeczy, których nie zrobiłbyś w kodzie produkcyjnym. Np. 500 wierszy w klasie.

Unit, plugin do grailsów:
|-pl.touk.excel.export [package]
|-getters [package]
|-AsIsPropertyGetter [public class]
|-Getter [public interface]
|-LongToDatePropertyGetter [public class]
|-PropertyGetter [public abstract?]
|-Formatters [public class]
|-WebXlsxExporter [public class]
|-XlsxExporter [public class]

Dużo jednolinijkowych metod. Plik z testami ma 400 wierszy. Tzn. że być może powinieniem rozbić to na odpowiedzialności (odpowiedzialność za wszystko jest słaba).

|-pl.touk.excel.export [package]
|-SampleObject.groovy [groovy]
|-XlsxExporterCreationTest [@Test]
|-XlsxExporterDateStyleTest [@Test]
|-XlsxExporterHeaderTest [abstract]
|-XlsxExporterManipulateCellsTest
|-XlsxExporterRowTest [@Test]
|-XlsxExporterTest [abstract]

Rozbicie na odpowiedzialności. Testuje zachowania np. wiersza, manipulujące komórką, tworzenie excela itd.
Może ten kod oryginalny też powinienem rozbić? Oczywiście.

|-pl.touk.excel.export [package]
|-abilities
|-CellManipulationAbility [public class]
|-FileManipulationAbility [public class]
|-RowManipulationAbility [public class]
|-getters [package]
|-AsIsPropertyGetter [public class]
|-Getter [public interface]
|-LongToDatePropertyGetter [public class]
|-MessageFromPropertyGetter [public class]
|-PropertyGetter [public abstract]
|-Formatters [public class]
|-WebXlsxExporter [public class]
|-XlsxExporter [public class]

Zasada 11: Nie testuj tylko dla testowania. Praktyka: nie zawsze wszystko się testuje.

Np. pisane AI w grach: postacie mają zachowywać się naturalnie. Napisał sobie w C# wizualizację AI. Tam dowiedział się jak to ma działać.

Prototypowanie: tam nie testujemy. Musimy być jak najbardziej kreatywani. Ale potem go KASUJEMY.

Czy potrzebujesz testować settery/gettery? Nie daje nam to bezpieczeństwa
Czy potrzebujesz testować biblioteki firm trzecich? Casami potrzebne, gdy nie ufamy.
Czy potrzebujesz testować anotacje JEE?

Uncle Bob: 100% pokrycia testami. Ale w domenie.
nie jest potrzebna baza danych

Zasada 12: Design. Bezpieczeństwo. Feedback. Komunikacja (co jest wymaganie). (testy są drogą do celu)

Zasada 13: Testy niepotrzebne należy kasować.
Kent Beck, "Test Driven Development by Example":
Kiedy kasować testy? Pierwsze kryterium to zaufanie do systemu (bezpieczeństwo). Nie usuwaj, jeśli redukuje poczucie bezpieczeństwa.
Drugie: komunikacja. Nie zapewnia bezpieczeństwa, ale mówi o wymaganiu.
Jeśli nie: usuwaj.
</pre>
</details>
---

## Wydajność<a name="wydajnosc"></a>

### Marcon Skalski: Wszystko co chcesz wiedzieć o wątkach, ale boisz się zapytać.<a name="skalski_watki"></a>

Bardzo szczegółowa i praktyczna wiedza.

<iframe width="560" height="315" src="https://www.youtube.com/embed/VklKR_fO5OI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<details>
    <summary>Moje notatki z prezentacji</summary>
<pre>

Implementacja wątku w Javie
 klasa Thread zawiera
  - priorytet wątku,
  - czy wątek jest demonem
  - jaki kod będzie uruchamiany na wątku w najbliższym czasie

Co się dzieje gdy tworzymy wątek thread.start();
- dostajemy stos, na który odkładane są ramki, waży ok 1MB (zależy od konfiguracji, wersji JVM, platformy)
- rejestr PC na którym znajdują się aktualnie wykonywane instrukcje (jeśli kod Jabva), jeśli kod natywny to zmienna niezdefiniowana
- fragment Edenu, tzw. TLAB (Thread Local Allocation Buffer), ciągły fragment Heap'u przeznaczony tylko dla naszego wątku. W momemcie, gdy alokojemy pamieć, tworzymy nowe obiektu nie musimy się martwić, że ktos nad zaczeni tu cos zapisywać.
- dostawiany jest wątek systemowy.

Wnioski: dużo pamięci bierze nowy wątek.

Głębiej w JVM: struct VM_Thread, która zawiera wskaźnik na klasę Java'ową i wskaźnik na wątek systemowy.
Wątek systemowy to lekkie procesy (lekkie bo większość informacji potrzebnych do działania dziedziczą po swoim rodzicu). Procesy opisane są w systemie jako deskryptory procesu. Zawiera on:
- stan, w którm znajduje sie proces
    - running - aktualnie wykorzystuje procesor lub czeka w kolejce
    - interuptable (uśpiony) - może zostać wybudzony za pomocą przerwania systemowego.
    - uninteruptable (uśpiony), jedyne co może wybudzić nas zproces, to operacja na której sie zablokował i na której śpi. Gdy zostahie zakończona to wróci do życia.
    - Gama statusów, gdy wątek przestał funkcjonować lub jest zamykany: STOPPED, TRACED, EXIT_ZOMBIE, EXIT_DEAD
- lista zadań, które mają być wykonane w najbliższym czasie
- wskaźniki na obszary z pamięcią wykorzystywane przez proces
- wskaźniki na rodzica
- wskaźniki na otwarte pliki/otwarte sockety
- listę sygnałów, które musi obsłużyć w najbliższym czasie.
  W systemie nasz proces jest identyfikowany za pomocą Process ID (PID). Kernel musi zamieniać PID na deskryptor.
  Kiedyś: lista deskryptorów,
  Teraz: tablica haszująca.

Process
==Stan RUNNING==
Gdy mamy jeden proces w systemie chciałby wykorzystać czas procesora to po prosrtu rejestry procesora są populowane danymi, które chciałby wykorzystać do swoich obliczeń. Oprócz tego populowane są cache procesora (L1). I wykonuje swoje obliczenia dopóki nie pojawi się inny proces, który też chciałby wykorzystać trochę czasy procesora. W tym momencie występuje ZMIANA KONTEKSTU. Wszystkie wartości, które były w rejestrach muszą zostac odstawione gdzieś na bok, a rejestry muszą zostać zpopulowane nowyi danymi.
Co się dzieje z Kontekstem sprzętowym (WG: czyli danymi w rejestrach?).
Część trafia do deskryptoru procesu, część na stos kernelowy.

Jak konkretnie wygląda populowanie rejestrów.

Kiedyś: instrukcja "far jump" - przeskakujemy do konkretne miejsca, które zawiera dane wykorzystywane przez następny proces.
Dzisiaj: Sekwencja strukcji "mov". Mamy kontrole jakie dane przenosimy i nie jest wolniejsza.
W przypadku populacji cache procesora. Problem każda kolejna warstwa cache'a jest wolniejsza. Nie mówiąc o doczytach z RAMu czy dysku. Gdy utworzymy dużą liczbę wątków czas przełączania się pomiędzy wątkami rośnie, bo procesor zamiast wykonywać obliczenia musi populować rejstrze, cache. A gdy jest więcje wątków, wtedy musi populowac z RAMu, co jest bardzo wolne i pali cykl procesora. Problem CONTEX SWITCHING.

Problem, który wątek ma dostać następny czas procesora: SCHEDULING.
W pierwszych wersjach schedulerów mieliśmy po prostu listę dwukierunkową procesów, kernel gdy zwalniał się procesor przechodził po liście i wybierał ten o najwyższym priorytecie. To rozwiązanie słabo się skaluje.
Kolejny algorytm w UNIX: O(1).
Mamy tablicę priorytetó, każda kamórka jest jednym priorytetem:
[Prio0, Prio1, Prio 2, ...]
W ramach prioryteów mamy listy, na którym znajdują się deskryptry procesów. Wybranie kolejnego procesu odbywa się w czasie stałym.

Kolejny algorytm:  Completely Fair Scheduler (CFS)
Operuje na drzewach czerwono czarnych, zamiast tablicy priorytetów. Przewaga nad O(1):
- O(1) ma problem: miał koncept EPOKI. W ramach każdej EPOKI kazdy proces musiał dostać swój czas procesora. W niektórych sytuacjach system mógł się wydawać nieresponsywany.

Dla każdego z tych algorytów mamy pełen zestwa Polices, które definiują w jaki sposób będzimy wybierać kolejny proces o danym priorytecie.
> Round Robin
Bierzemy po prosru pierwszy proces z kolejki, gdy zużyje swój kwant czasu, idzie na koniec kolejki.
> FIFO
Proces o najwyższym priorytecie dostaje tyle czasu ile potrzebuje. Gdy pojawi się proces o wyższym  priorytecie dostaje on czas procesora, dopóki nie skończy sowich obliczeń.
> Normalny
Kernel na bieżąco przechodzi sobie po procesach, wylicza dynamicznie priorytety, w momencie gdy zakończy się kwant czasu, wybiera kolejy proces o najwyższym priorytecie.

Jaka konfiguracja dla nas?
Przeliczając te paczki będzie wykonywał operacje BATChowe, czyli będzie czekał na operacje wejścia/wyjścia.
Najlepiej byłoby wybrać taką ilość wątków, jaka liczba procesorów w systemie. Bo w pełni będą się wykonywać.
Czyli CFS + FIFO. Ponieważ chcemy, żeby nasze batche jak najszybciej się policzyły, nie zależy nam na interaktywaności naszego systemu.
[22:40]
</pre>
</details>

---

## Inne bazy wiedzy<a name="inne_bazy_wiedzy"></a>

### Bottega<a name="bottega"></a>

Firma szkoleniowa skupiająca najlepszych praktyków programowania w Polsce udostępnia za darmo wartościowe materiały. 

[bottega.com.pl/materialy-Java](https://bottega.com.pl/materialy-Java)

Przy okazji polecam udostępniony przez nich za darmo 

> Darmowy kurs online dla programistów i programistek na poziomie junior i regular

[devupgrade.online](https://devupgrade.online/)

### Krzysztof Góralski: baza wiedzy<a name="kgoralski"></a>

"Personal" wiki, ale publiczne, więc myślę, że Krzysztof nie będzie miał nic przeciwko udostępnieniu tutaj linku do jego wiki: 

[kgoralski.gitbook.io/wiki](https://kgoralski.gitbook.io/wiki/)

Świetne zgrupowanie materiałów — inspiracja na rozwinięcie tego wpisu :)



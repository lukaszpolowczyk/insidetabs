# Tabs API or Website Context API - kontrola wielu kart przez jedną instancję strony internetowej

[ dokument niegotowy ]

## Wstęp

Proponowane API Kart jest w dużym stopniu analogiczne i uzupełniające dla History API.  
Tabs API rozszerzy funkcje zawarte w History API na karty tego samego pochodzenia(SOP) co umożliwi zintegrowaną kontrolę nad nimi, zmniejszenie objętości stron i doskonałą synchronizację między kartami.  
Tak jak History API może przejąć przyciski wstecz/do przodu przeglądarki. Tabs API może przejść karty przeglądarki.  
Nowe API jest proste w implementacji i nie komplikuje życia użytkownika.

## Cele

Przy wielu otwartych kartach tego samego pochodzenia:

* Strona internetowa zajmować będzie mniej miejsca.
* Strona internetowa będzie miała mniejsze zużycie procesora.
* Strona internetowa będzie się znacznie szybciej uruchamiała.
* Sprawniejsze przełączanie między kartami.
* Doskonała synchronizacja stałych elementów strony, jak np. czat, powiadomienia na Facebooku.
* Lepsze zarządzanie stroną dla developera (brak potrzeby obsługi nietypowych sytuacji, np. nadpisywania stanów tych samych elementów)
* łatwiejszy dostęp do informacji dla developera, o sposobie używania strony przez użytkownika, zachowanie ciągłości używania strony
* Czasami strona działa prawidłowo tylko pod jedną postacią. Zdarza się, że strona otwarta w nowej karcie nie otwiera się, tylko zwraca informację że może działać tylko w jednej karcie (strona PKP). to API umożliwi, żeby w każdej nowej karcie mogła otwierać się dokładnie ta sama strona, nawet bez zmiany jakiegokolwiek elementu na niej. Ewentualnie można dodać API przechwytujące nową stronę i nie otwierajacą jej, tylko wracającą do już otwartej - istnieje już takie coś? Dla rozszerzeń chyba istnieje.


## Nazewnictwo

* Browser tab - karta przeglądarki internetowej, element UI.
* Inside Tab - karta (element UI przeglądarki) kontrolowana przez instancję stony internetowej.
* Website instance -  pojedyncze wystąpienie niezależnego kodu zgodnego z danym wzorcem. Instancja strony internetowej (kodu HTML, CSS, JS etc.), czyli niezależnych bytów danej strony internetowej, zajmującej określone miejsce w pamięci, uruchomionej w przeglądarce internetowej.
* Instance snapshot - stan strony internetowej w danym kontekście
* Website (Browser) Context - kontekst przeglądarki w jakiej "jest" strona internetowa. Kontekstem może być Tab, WebExtensions Popup, Sidebar, IFrame, Window. Pojęcie ogólniejsze od Inside Tab.


## Stan obecny

Aktualnie jedna karta przeglądarki internetowej jest powiązana z jedną instancją strony internetowej:

![One tab - one Instance of website](onetaboneinstance.png)

Przy czym np. V, X i Y mogą być instancjami tej samej strony, tylko np. innej podstrony.

Klikniecie Browser Tab powoduje załadowanie strony i wyświetlenie jej do widoku przeglądarki.

### Przykład stanu obecnego

Trzy karty zawierajace różne podstrony Facebooka. Trzy filmy z YouTube otwarte w trzech osobnych kartach.


## Propozycja

Wiele kart może być powiązane z jedną instancją strony internetowej:

![Many tabs - one Intance of website](manytabsoneinstance.png)

Kliknięcie Browser Tab powoduje załadowanie strony i wyświetlenie jej widoku oraz wywołanie `newcontext` event. Ale gdy otwarta już jest np. First Tab i użytkownik przejdzie do Second Tab to strona już się nie ładuje ani nie wyświetla od nowa(zostaje tak jak jest), a tylko wywołuje się `newcontext` event i przekazuje do callback-u informacje o kontekście.
 and transmit context to this Instance though event `newcontext`

To deweloper strony decyduje czy przechowywać treść wszystkich kart na raz w pamięci, czy oczyszczać pamięć i ładować dopiero po przejściu do danej karty. Jest to w pełni pod kontrolą dewelopera i może korzystać z dowolnych rozwiązań.  

Przełączanie między kartami zmienia tylko kontekst(adres, tytuł itp.) tej samej instancji strony oraz wywołuje event zwracający kontekst.

Przy przełączaniu między kartami z adresem tego samego pochodzenia strona zostaje ta sama, zmienia się tylko [końcówka adresu] i tytuł, tak jak przy Hostory API, gdy klika się przycisk Wstecz.

## API

### Event `newcontext/opencontext/startcontext`, `closecontext/endcontext` i `changecontext`

```javascript
navigator.addEventListener("newcontext", (context)=>{ // id do rozróżniania kontekstów
   // aktywuje się, gdy otworzy się nową kartę z adresem strony - np. poprzez kliknięcie linka kółkiem, menu kontekstowe i Otwórz link w nowej karcie/oknie
   
});
```

#### Event `newcontext` aktywuje się gdy

* user kliknie w link tego samego pochodzenia (linka klikniętego w tej samej ale też w innej domenie - instancja strony przejmuje każdy link)
* wywoła w każdy sposób nowe okno/kartę (w tym z poziomu paska adresu, paska zakładek, dodatku, z zewnętrznej aplikacji (być może ze względów bezpieczeństwa lepiej to zablokować ale w zasadzie i tak event to przejmuje wiec raczej nie powinno być porblemów, można odrzucić jakieś dziwne adresy))
* nie jestem przekonany czy powinien się aktywować gdy pierwszy raz otworzy się instancję strony (pierwszy kontekst tej instancji), czy stworzyć osobny event do tego, czy może nawet nie będzie potrzebny, bo po prostu można skorzystać z już istniejących event-ów.

Parametr context zawiera adres (czyli tylko(?) adres strony jest kontekstem). Chociaż zastanawiałem się czy można zezwolic na wiele kart z tym samym adresem w tym samym kontekście (wtedy dana karta z danym adresem musiałaby przejmować wszelkie próby otwarcie nowej karty z tym adresem, lub tworzyć wtedy nową instancję).
Parametr context zawiera też jakieś informacje o źródle otwartego adresu.

## Zachowanie przeglądarki

* Gdy użytkownik użyje (standardowej w przeglądarkach) funkcji duplikacji strony, powinna zostać wymuszona osobna instancja strony. Wynika to z założenie że inaczej funkcja duplikacji nic by nie zrobiła, prócz tworzenia karty z tym samym kontekstem. Chociaż być może istnieją sytuacje gdzie dwie karty z tym samym kontekstem tej samej instancji będą potrzebne?
* Być może niektóre Event-y strony będą działać nietypowo, jak np. event wejścia i wyjścia stronę gdy przełącza się między kartami tej samej instancji. W takiej sytuacji event wyjścia ani wejścia nie powinien się uaktywniać. Być może będzie potrzeba powstania dodatkowych event-ów obsługujących przejście między kontekstami tej samej instancji strony (zaraz, przecież to ma być chyba podstawowa funkcja Tabs API?!). Notuję to, żeby zwrócić uwagę że w procesie dostosowania stron do Tabs APi będzie być może konieczne skorzystanie z obu rozwiązań.
* Zmiana stanu już wskazanej karty?? a może to history api wystarczy. replaceState. Jak się ma History API do Tabs API? - czy historia jest wspólna dla każdej karty? Raczej tak być nie powinno, czyli każdy kontekst musi mieć swoją historię.


### Przykład użycia propozycji

React Router

Zdarza się, że otwiera się tą samą stronę w wielu kartach, te karty zajmują tyle co cała strona, ale różnice między kartami to np. tylko ten obrazek i kilka komentarzy pod nim. Więc realną korzyścią jest znaczne zmniejszenie obciążenia pamięci, ale też procesora, w sytuacjach otwierania wielu kart tej samej strony.


## Polyfill

Da się utworzyć prosty polyfill dla API kart, tworząc pasek kart wewnątrz strony. Jednak jest to rozwiązanie znacznie gorsze niż natywne Tabs API.

**Polyfill vs Native solution**:

|Polyfil | Native solution|
|:------:|:--------------:|
| Karty zajmują dodatkowe miejsce w pionie | Karty nie zajmują dodatkowego miejsca w pionie |
| Pasek kart zachowuje się nieprzewidywanie, to jest - tak jak go zaprojektują deweloperzy strony na której on się znajduje | Pasek kart zachowuje się zawsze tak samo, przewidywalnie, zgodnie z przyzwyczajeniem - tak jak zaprojektują to deweloperzy przeglądarek |
| Można otworzyć część kart w pasku przeglądarki a część w pasku Polyfill-a na stronie co może spowodować bałagan i utrudnić dostęp do części kart | Można otworzyć karty tylko w pasku przeglądarki, dzięki temu zachowany jest porządek i bezpośredni dostęp do kart |


## Animacja przesuwania div-a z kartami ze strony do postaci kart w przeglądarce

## Wątpliwości

* konieczność dostosowania się do nowego API - nie ma takiej konieczności. można wymiennie i nawet jednocześnie z poziomu tej samej strony, wedle potrzeb, używać i standardowego rozwiązania i Tabs API. Dla użytkownika powinno to być w zasadzie niezauważalne (nie licząc braku korzyści z używania Tabs API) ani utrudniać procesu dewelopingu.
* Cache API jako optymalizacja ładowania strony? W porównaniu do Tabs API jest o wiele gorsze bo Tabs API nie musi przetwarzać ponownie całej strony.
* Message API do synchronizacji? Raczej nieporównywalnie gorsze i bardziej skomplikowane rozwiązanie.

- przeładowywanie skryptu, zamiast po prostu załadowanie już gotowej strony - może opóźnienia wystąpią, ale developer ma pełną kontorlę nad tym, może tego użyć optymalnie, może też tego API po prostu nie użyć, jeśli np. i tak cała strona musi się załadować od nowa i nie ma części wspólnych.
- jeśli jakaś strona tego potrzebuje, to niech po prostu zrobi własny pasek wewnątrz strony, do przełączania między kartami i tam kontroluje przełączanie między kartami - może tak być, ale jednak prawie każdą stronę można otwierać w nowej karcie i prawie nikt tego nie ogranicza, a optymalizacja z powodu tego API jest całkiem realna dla każdej strony, którą otwiera się w wielu kartach przeglądarki, nie tylko dla specjalnych stron typu aplikacja, która może posiadać wewnętrzny system kart. A jeśli użyć
- pomieszanie stron - to dotyczy też każdej strony, a jednak użytkownicy używają w ten sposób przeglądarki, więc brak tego API tego nie rozwiąże, a istnienie tego API rozwiąże inną kwestię (optymalizacji)
- to APi nie rozwiązuje tych problemów, bo i tak musi istnieć wersje stron, które ładuję się jako osobna strona, jako wsparcie dla starszych przeglądarek oraz gdy użytkownik nie zezwoli na używanie InsideTabs. Faktycznie, i tak trzeba przygotować takie wersje strony, ale w jakimś stopniu to i tak pomaga, a poza tym, głównym celem jest mniejsza zajętość RAM-u i procesora, łatwiejsze uzyskanie optymalizacji.
- istnieje onMessage, które umożliwia przekazywanie pewnych rzeczy. Nadal nie jest to tak proste i wygodne jak InsideTabs.

## Kontrola przez użytkownika

Z racji, że Tabs API jest wymienne do standardowego otworzenia nowej karty, użytkownik będzie miał możliwość wyłączenia Tabs API dla strony lub dla całej przeglądarki za pomocą standardowego Permissions używane do różnych uprawnień w przeglądarce. Tabs API jednak może być bez przeszkód domyślnie włączone w przeglądarkach.


## Dodatkowe


* API do kontrolowania kart - możliwość zamykania kart (tak jak istnieje możliwość zamykania  okna z poziomu strony), API dające możliwość przełączania się między kartami tej samej strony(w zasadzie poprzez open(adres) to powinno zadziałać, poprzez "przejmowanie" karty). Przesuwanie kart już sobie chyba odpuścimy, bo to tak jakby gubić kontekst.
* co z mobile - to samo - co z pwa? chyba to samo?

Pomyśleć o połączeniu InsideTabs w jedną rozgałęzioną(?) historię kart. Nie koniecznie, bo historia powinna też dotyczyć innych domen, otwieranych przez siebie. Ale już historia "karty", czyli przycisk wstecz/do przodu to już mogą być w odpowiedni sposób połączone - np. przycisk wstecz w otwartej nowej karcie z karty z tej samej domeny, może zamknąć tą kartę i przejść automatycznie do źródłowej(albo bez zamykania), a tak zamkniętą kartę oczywiście można przywrócić przeglądarkową funkcją przywracania.

Dodatkowe API, mające w jakimś stopniu zastąpić już istniejące wewnętrzne systemy kart:
- tabContextMenuItems - dodatkowe funkcje dla kart dla strony w menu kontekstowym karty
- OpenManTabsThisSameDomain - otwiera kilka kart z tą samą domeną (coś jak przywracanie sesji, ale dla konkretnej strony i robione nie przez przeglądarkę ani user-a, tylko przez stronę)
- przejście na konkretną stronę z zestawu kart tej samej domeny będących InsideTabs za pomocą skryptu. Od razu do tego zabezpieczenie, żeby skrypt nie trollował i nie otworzył wielu kart po czym przeskakiwał między nimi - ograniczenie przełączania kart na minutę, jakieś rozsądne.
- cross-domain - np. cukierberg może połączyć domenę facebooka z instagramem i wtedy wspólnie kontrolować te dwa serwisy, jakby były jedną stroną/aplikacją, zrobiłoby to taką wyrwę w przeglądarce, wspólnej przestrzeni dla jakiegoś szerszego środowiska, które by ze sobą współgrało w wielu kartach (a nawet w jednej karcie po sobie - wymaga połączenia InsideTabs z history API) i byłoby wydajniejsze, sprawniejsze przełączanie między sobie, coś jakby istniała specjalna przeglądarka cukierberga dostosowana pod jego serwisy (tak samo każde inne środowisko czy to Google czy MS), a jednocześnie w żaden sposób nie naruszałoby to kontroli przez przeglądarkę i przez użytkownika, po prostu dawałoby pewne korzyści przy używaniu spójnej platformy jednego właściciela.
- można by w niektórych przypadkach korzystać z tego tak, że tworzy się new window i podmienia stary window na nowy, pozostając na "tej samem" stronie, a jednocześnie podmieniać w niektórych przypadkach tylko fragmenty strony. Pełna dowolność. Oczywiście otwieranie nowej czystej karty też byłoby możliwe dla developera.

API kontroli wielu kart przez jedną instancję "strony" (SORS).

prosty polyfill z paskiem kart nad stroną i działajacym samym w sobie API.

Nazewictowo:
* Brower tab - karta przeglądarki internetowej, element UI.
* InsideTab - karta (element UI przeglądarki) kontrolowana przez instancję stony internetowej.
* Website instance -  pojedyncze wystąpienie niezależnego kodu zgodnego z danym wzorcem. Instancja strony internetowej (kodu HTML, CSS, JS etc.), czyli niezależnych bytów danej strony internetowej, zajmującej określone miejsce w pamięci, uruchomionej w przeglądarce internetowej.
* Instance snapshot - stan strony internetowej w danym kontekście

Aktualnie jedna instancja strony jest powiązana z jedną kartą przeglądarki internetowej.

![One tab - one Instance od website(onetaboneinstance.png)
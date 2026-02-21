## Architecture
- Kubernetes follows a control plane-worker node architecture
- Control Plane (the brain): makes global decisions and manages the cluster's state.
- Worker Nodes (The Muscle): run the actual application workloads.
#### The Control Plane (The Brain)
- kube-apiserver: the central hub and frontend. all communication goes thrugh the API server.
- etcd: The cluster's single source of truth; a consistent, distributed key-value store for all cluster data.
- kube-scheduler: the matchmaker. Watches for new Pods and assigns them to the best Node.
- kube-controller-manager: the autopilot. Runs various controller processes (Node Controller., Replication Controller, etc.) to maintain the desired state.
#### Worker Nodes
- kubelet: the primary "node agent". Ensures containers described in Pods are running and healthy. Manges the container lifecycle on its node.
- kube-proxy: a network proxy on each node. Maintains network rules to allow commmunication to your Pods (the basis of the Service concept).
- Conatiner runtime: the software that runs containers (e.g., containerd, CRI-o). The kubelet communicates with it via the Container Runtime Interface (CRI). 
## The SSH "Node-Hopping" Rule
- assume you must move: most tasks require switching from the base node to a worker or control plane node.
- Verify Your Prompt: Always double-check user@node before executing commands.
- THe Power Move: Use sudo -i immediately after SSH-ing to fain root access.
- Essential for: Editing kube-apiserver.yaml and other system files.
## Optimizing the Browser
- The Two-Tab Rule:
  - Exam Environment
  - Official Documentation (kubernetes.io/docs)
- Search Strateties: Use the internal site search for rapid access to Gateway API or NetworkPolicy templates.
- Manual Navitgation: Don't rely on personal bookmarks;
  they won't be there. Know the doc structure by heart.



PL:
Mówiąc najprościej: Control Plane (warstwa sterowania) to „mózg” klastra Kubernetes. To tutaj zapadają wszystkie decyzje dotyczące tego, co ma się dziać w systemie, jak mają być rozmieszczone aplikacje i jak reagować na awarie.

Oto kluczowe elementy, które składają się na to centrum dowodzenia:
Kluczowe komponenty Control Plane
1. **kube-apiserver** To „okienko podawcze” klastra. Każda komenda, którą wpisujesz w kubectl, przechodzi właśnie tędy. To jedyny komponent, który rozmawia bezpośrednio z bazą danych klastra.
2. **etcd** To pamięć klastra. Bardzo szybka i niezawodna baza danych (typu klucz-wartość), w której przechowywane są wszystkie informacje o stanie systemu: co działa, gdzie i z jakimi parametrami.
3. **kube-scheduler** To logistyk. Kiedy pojawia się nowa aplikacja (Pod) do uruchomienia, scheduler sprawdza zasoby na dostępnych maszynach (Worker Nodes) i decyduje, gdzie ją „posadzić”, biorąc pod uwagę m.in. dostępne RAM i CPU.
4. **kube-controller-manager** To strażnik porządku. Składa się z wielu kontrolerów, które dbają o to, by stan faktyczny zgadzał się ze stanem pożądanym. Na przykład: jeśli zadeklarowałeś, że mają działać 3 kopie aplikacji, a jedna padnie, kontroler to zauważy i wyda polecenie uruchomienia nowej.
5. **cloud-controller-manager** (opcjonalnie)
Łącznik z chmurą (np. AWS, Azure, GCP). Pozwala Kubernetesowi zarządzać zasobami specyficznymi dla dostawcy, jak load balancery czy dyski sieciowe.

Jak to działa w praktyce?
Wyobraź sobie Control Plane jako wieżę kontroli lotów:
- **Ty** wysyłasz plan lotu (plik YAML z konfiguracją).
- **API Server** odbiera plan i zapisuje go w etcd.
- **Scheduler** znajduje wolny pas startowy (Worker Node).
- **Controller Manager** pilnuje, aby samolot nie tylko wystartował, ale i pozostał w powietrzu.
    **Warto wiedzieć**: W środowiskach produkcyjnych Control Plane zwykle działa na kilku maszynach naraz (High Availability), aby nawet po awarii jednego „mózgu” klaster nadal wiedział, co robić.

Skoro Control Plane to mózg klastra, to Worker Nodes są jego „mięśniami”. To właśnie na tych maszynach (fizycznych lub wirtualnych) realnie działają Twoje aplikacje wewnątrz kontenerów.
Gdy Control Plane zdecyduje, że aplikacja ma zostać uruchomiona, wysyła instrukcje do konkretnego Worker Node'a, a on zajmuje się resztą.
Z czego składa się Worker Node? 
Każdy węzeł roboczy musi posiadać trzy kluczowe komponenty, aby móc dogadać się z „mózgiem” i zarządzać kontenerami:
1. **kubelet**
   To najważniejszy agent na każdym węźle. Możesz go sobie wyobrazić jako „brygadzistę”. Otrzymuje on instrukcje od Control Plane i pilnuje, aby kontenery opisane w specyfikacji (PodSpec) faktycznie działały i były zdrowe.
2. **kube-proxy**
   To strażnik ruchu sieciowego. Odpowiada za to, aby pakiety trafiały tam, gdzie powinny – obsługuje reguły sieciowe na hoście i umożliwia komunikację między usługami (Services).
3. **Container Runtime**
   To silnik, który fizycznie uruchamia kontenery. Kubernetes nie robi tego sam – potrzebuje oprogramowania trzeciego, takiego jak containerd czy CRI-O (kiedyś najpopularniejszy był Docker).
Porównanie: Control Plane vs. Worker Nodes
Cecha			| Control Plane (Mózg)		|			Worker Node (Mięśnie)
-----------|-----------------------|--------------------------------
Główna rola		|	Zarządzanie i podejmowanie decyzji.	|	Uruchamianie aplikacji (kontenerów).
Kluczowe dane	|	Przechowuje stan całego klastra (etcd). |	Przechowuje logi i dane działających aplikacji.
Liczba		|		Zazwyczaj 1-3 (dla redundancji).	|	Od jednego do tysięcy (skalowalność).
Interakcja		|	Rozmawia z użytkownikiem (kubectl).	|	Rozmawia z Control Plane (przez kubelet).

Co się stanie, jeśli Worker Node padnie? 
To jest moment, w którym Kubernetes pokazuje swoją magię. Jeśli maszyna (Worker Node) ulegnie awarii: 
1. **Control Plane** zauważy brak sygnału od kubeleta.
2. **Controller Manager** uzna, że kontenery na tym węźle są stracone.
3. **Scheduler** znajdzie inny, zdrowy Worker Node.
4. Aplikacje zostaną automatycznie uruchomione na nowym węźle, aby utrzymać zadeklarowaną liczbę kopii. 

**Pod** to najmniejsza i najprostsza jednostka w Kubernetesie, jaką możesz stworzyć lub wdrożyć. Możesz o nim myśleć jak o „opakowaniu” dla Twojego kontenera.

**Dlaczego Pod, a nie po prostu Kontener?**
W świecie Dockera uruchamiasz kontener. W Kubernetesie kontener zawsze musi znajdować się wewnątrz Poda. Dlaczego? Ponieważ Pod daje kontenerom dodatkowe supermoce:
- **Wspólne zasoby**: Wszystkie kontenery wewnątrz jednego Poda dzielą ten sam adres IP oraz te same zasoby dyskowe (wolumeny).
- **Bliska współpraca**: Kontenery w Podzie mogą rozmawiać ze sobą przez localhost. To tak, jakby mieszkały w jednym pokoju.
- **Jednostka skali**: Kubernetes nie skaluje pojedynczych kontenerów, tylko całe Pody. Jeśli potrzebujesz więcej mocy, uruchamiasz kolejną kopię Poda.

**Anatomia Poda: Jeden czy wiele kontenerów?**
W 90% przypadków będziesz stosować model "One-container-per-Pod". Jednak czasem zdarza się, że główna aplikacja potrzebuje "pomocnika". Takie dodatkowe kontenery nazywamy:
1. **Sidecar containers**: Pomagają głównej aplikacji, np. przesyłając logi do centralnego serwera lub odświeżając certyfikaty SSL.
2. **Init containers**: Uruchamiają się przed główną aplikacją, np. żeby sprawdzić, czy baza danych jest już gotowa do przyjęcia połączeń.

**Jak wygląda życie Poda?**
Pody są epimeryczne (tymczasowe). To bardzo ważna koncepcja!
Jeśli Pod zginie (np. braknie RAM-u na serwerze), Kubernetes go nie "wskrzesza". Zamiast tego **Controller Manager** zauważa stratę, a **Scheduler** tworzy zupełnie nowy egzemplarz Poda (z nowym adresem IP) na innym (lub tym samym) węźle.
  Złota zasada: Nigdy nie przywiązuj się do konkretnego Poda. Pody to „bydło, a nie zwierzątka domowe” (Cattle, not Pets) – jeśli jeden zachoruje, po prostu zastępujemy go nowym. 

**Jak to wszystko łączy się w całość? (Podsumowanie)**
1. Ty wysyłasz plik YAML do **Control Plane** (API Server).
2. **Control Plane** (Scheduler) sprawdza, który Worker Node ma wolne miejsce.
3. **Worker Node** (Kubelet) otrzymuje rozkaz i uruchamia Poda.
4. Wewnątrz **Poda** rusza Twój **kontener** z aplikacją.

Aby użytkownik nie stracił danych, a system działał płynnie mimo „śmierci” Poda, Kubernetes stosuje dwa oddzielne mechanizmy: jeden dla **danych**, a drugi dla **ruchu sieciowego**.

1. **Dane nie giną dzięki Wolumenom** (Persistent Volumes)
Domyślnie system plików wewnątrz kontenera jest tymczasowy. Jeśli Pod zginie, wszystko, co zapisał „u siebie” na dysku, znika. Aby temu zapobiec, używamy zewnętrznej pamięci.
- **Persistent Volume** (PV): To kawałek dysku (np. w chmurze AWS EBS, dysk Google, czy sieciowy NFS), który istnieje niezależnie od Poda.
- **Persistent Volume Claim** (PVC): To „zlecenie” od Poda, który mówi: „Potrzebuję 10GB trwałego miejsca”.
Gdy Pod umiera, nowy Pod (stworzony przez Kubernetes) „podpina się” pod ten sam dysk (PV). To tak, jakbyś wyciągnął pendrive'a z zepsutego laptopa i włożył go do nowego – Twoje pliki tam są.

2. Użytkownik nie zauważa przerwy dzięki **Serwisom** (Services)
Skoro nowy Pod dostaje **nowy adres IP**, to skąd Twoja przeglądarka wie, gdzie go szukać? Tutaj wkracza Service.
- **Service** to stały punkt kontaktowy (stałe IP i nazwa DNS).
- Działa jak inteligentny drogowskaz: Ty łączysz się z Serwisem, a on przekazuje ruch do aktualnie żyjących Podów.
- Jeśli jeden Pod zginie, Service natychmiast przestaje wysyłać tam ruch i kieruje go do nowo utworzonego Poda. Dla użytkownika to zazwyczaj milisekundowa przerwa, często niezauważalna.

3. **A co z danymi "w locie"** (np. wypełniany formularz)?
Tu musimy być szczerzy: Kubernetes dba o infrastrukturę, ale nie o logikę Twojej aplikacji.
- **Bazy danych**: Jeśli dane zostały już zapisane w bazie (która korzysta z Persistent Volumes), są bezpieczne.
- **Sesje użytkownika**: Jeśli Twoja aplikacja trzyma informację o zalogowaniu tylko w pamięci RAM Poda, to po jego śmierci użytkownik zostanie wylogowany.
- **Rozwiązanie**: Deweloperzy trzymają takie dane w zewnętrznych magazynach, np. w **Redis**, aby każdy nowy Pod mógł z nich skorzystać.

Problem | Rozwiązanie w Kubernetes
---------|-------------------------
Gdzie są moje pliki? | "Persistent Volumes (dysk zewnętrzny, który „przeżywa” Poda)."
Gdzie wysłać zapytanie? | "Service (stały adres, który zawsze wie, gdzie są żywe Pody)."
Kto pilnuje porządku? | Deployment/StatefulSet (automatycznie tworzą nowego Poda w miejsce starego).

To trochę jak z kinem: jeśli zepsuje się projektor (Pod), obsługa szybko przenosi widzów do innej sali (nowy Pod). Film (dane na PV) jest puszczany od tego samego momentu, a bilet (Service) nadal jest ważny.

**Czym różni się Deployment (dla zwykłych apek) od StatefulSet (dla baz danych)?**
1. Deployment: „Byle ich było trzech”
Deployment służy do aplikacji bezstanowych (stateless). To najczęstszy sposób wdrażania aplikacji (np. frontend w React, mikroserwis w Javie czy Pythonie).
- **Podejście**: Kubernetes traktuje te Pody jak wymienne części (identyczne kopie).
- **Nazewnictwo**: Pody dostają losowe nazwy (np. web-6d4cfd-abc12).
- **Dane**: Jeśli jeden Pod zginie, nowy nie przejmuje „osobistych” plików poprzednika. Wszystkie Pody w Deploymencie zazwyczaj patrzą na to samo, wspólne źródło danych (np. tę samą bazę).
- **Główny cel**: Skalowanie i dostępność. Jeśli potrzebujesz więcej mocy, po prostu dopisujesz replicas: 10.

2. StatefulSet: „Każdy ma swoje imię i swój dysk”
**StatefulSet** jest przeznaczony dla aplikacji stanowych (stateful), takich jak bazy danych (PostgreSQL, MongoDB, Cassandra) czy kolejki (Kafka).
- **Podejście**: Każdy Pod jest unikalny i ma swoją tożsamość, która zostaje zachowana nawet po restarcie.
- **Nazewnictwo**: Pody mają stałe numery (np. db-0, db-1, db-2). Zawsze uruchamiają się w tej samej kolejności.
- **Dane** (Kluczowa różnica!): Każdy Pod w StatefulSet ma swój własny, przypisany na stałe dysk (Persistent Volume).
- Jeśli db-1 padnie, Kubernetes stworzy nowego Poda o nazwie db-1 i podepnie mu dokładnie ten sam dysk, z którego korzystał wcześniej. Dzięki temu baza danych "pamięta", co zapisała przed awarią.

**Porównanie w pigułce**
Cecha | Deployment (Stateless) | StatefulSet (Stateful)
------|------------------------|------------------------
Przeznaczenie | "Frontend, API, Mikroserwisy" | "Bazy danych, Kolejki, Systemy plików"
Nazwa Poda | Losowa (np. app-x5z2),Stała i przewidywalna (db-0)
Zarządzanie dyskami | Często wspólny dysk lub brak | Indywidualny dysk dla każdego Poda
Kolejność startu | Wszystkie naraz | "Jeden po drugim (0, potem 1...)"

**Dlaczego to jest ważne dla Ciebie?**
Jeśli tworzysz prostą stronę internetową, używasz Deploymentu. Jest szybki, łatwy do aktualizacji (tzw. Rolling Updates) i wybacza błędy.
Jeśli jednak konfigurujesz klaster bazy danych, musisz użyć StatefulSet. Bez tego, po restarcie Poda, Twoja baza danych mogłaby „obudzić się” z pustym dyskiem lub podpiąć się pod dysk innej instancji, co skończyłoby się katastrofą (korupcją danych).
  **Ciekawostka**: W nowoczesnych architekturach dąży się do tego, aby jak najwięcej aplikacji było stateless (Deployment), a bazy danych (Stateful) często deleguje się do gotowych usług chmurowych (np. AWS RDS), żeby nie musieć ręcznie zarządzać skomplikowanymi StatefulSetami.

**Jak w praktyce aktualizuje się taką aplikację bez przerwy w działaniu (tzw. Zero Downtime Deployment)?**
Dzięki mechanizmowi **Rolling Update** możesz wdrażać nową wersję aplikacji (np. v2.0 zamiast v1.0) w taki sposób, że użytkownicy nawet nie zauważą zmiany.
Oto jak Kubernetes robi to "po cichu" pod maską:

**Strategia Rolling Update (Krok po kroku)**
Wyobraź sobie, że masz Deployment z 3 replikami Twojej aplikacji w wersji v1. Chcesz wgrać wersję v2.
1. **Start nowego Poda:** Kubernetes nie zabija od razu starych wersji. Najpierw uruchamia jeden nowy Pod z wersją v2.
2. **Sprawdzenie gotowości (Readiness Probe):** To kluczowy moment. Kubernetes czeka, aż nowy Pod powie: „Jestem gotowy, baza podpięta, możecie wysyłać mi ruch”.
3. **Przełączenie ruchu:** Gdy v2 jest gotowy, Service zaczyna kierować do niego zapytania od użytkowników.
4. Usunięcie **starego Poda**: Dopiero teraz Kubernetes zabija jeden Pod wersji v1.
5. **Powtórka**: Proces powtarza się, aż wszystkie Pody v1 zostaną zastąpione przez v2.

**Co jeśli nowa wersja ma błąd? (Self-healing)**
To jest sytuacja, w której Kubernetes ratuje Ci skórę. Jeśli Twoja wersja v2 ma krytyczny błąd i nie przejdzie testu gotowości (**Readiness Probe**):
- Kubernetes **zatrzyma aktualizację**.
- Nowy Pod nigdy nie zostanie uznany za "zdrowy", więc **Service** nie wyśle do niego żadnego użytkownika.
- Stare Pody (v1) nadal działają i obsługują ruch.
- Twoi klienci są bezpieczni, a Ty masz czas na poprawienie błędu w kodzie.
  **Protip**: Jeśli jednak aktualizacja przejdzie, ale po minucie zauważysz błędy w logach, możesz wpisać jedną magiczną komendę: kubectl rollout undo deployment/moja-apka. Kubernetes błyskawicznie cofnie zmiany do poprzedniej,      działającej wersji.

Dwie najważniejsze liczby: **MaxSurge** i **MaxUnavailable**
W konfiguracji Deploymentu możesz precyzyjnie określić, jak agresywny ma być ten proces:
- **MaxSurge**: O ile więcej Podów może istnieć ponad limit w trakcie aktualizacji? (Np. „pozwalam na 1 nadprogramowy Pod, żeby przyspieszyć proces”).
- **MaxUnavailable**: Ile Podów może być niedostępnych w tym samym czasie? (Np. „zawsze muszą działać przynajmniej 2 kopie z 3”).

**Podsumowanie: Dlaczego warto?**
Cecha | Tradycyjne wdrożenie | Kubernetes (Rolling Update)
------|----------------------|-----------------------------
Dostępność | Przerwa w działaniu (Downtime) | 100% dostępności (Zero Downtime)
Bezpieczeństwo | Trudny powrót do starej wersji | Błyskawiczny _rollout undo_
Automatyzacja | Ręczne przełączanie serwerów | Pełna automatyzacja przez Control Plane

Powtórka:

1. **Control Plane** (Mózg)
To centrum decyzyjne klastra. Składa się z:
- API Server: Punkt kontaktowy dla użytkownika.
- etcd: Baza danych o stanie klastra.
- Scheduler: Decyduje, gdzie uruchomić kontenery.
- Controller Manager: Pilnuje, aby stan faktyczny zgadzał się z planem.
2. **Worker Nodes** (Mięśnie)
Maszyny, na których działają aplikacje:
- **Kubelet**: Agent wykonujący polecenia Control Plane.
- **Kube-proxy**: Zarządza ruchem sieciowym.
- **Container Runtime**: Silnik (np. containerd) uruchamiający kontenery.

3. **Pod** (Podstawowa jednostka)
Najmniejszy obiekt w Kubernetesie. Grupuje jeden lub więcej kontenerów, które dzielą IP i wolumeny dyskowe. Pody są tymczasowe – jeśli zginą, są zastępowane nowymi.

4. **Trwałość danych i dostępność**
- **Persistent Volumes** (PV): Zewnętrzne dyski, które nie giną wraz z Podem.
- **Services**: Stałe adresy IP, które kierują ruch do właściwych Podów, nawet jeśli ich IP się zmienia.

5. **Deployment vs. StatefulSet**
- **Deployment**: Dla aplikacji bezstanowych (identyczne kopie, np. frontend).
- **StatefulSet**: Dla baz danych (każdy Pod ma stałą nazwę i własny, unikalny dysk).

6. **Zero Downtime Deployment** (Rolling Update)
Proces aktualizacji, w którym Kubernetes stopniowo wymienia stare Pody na nowe (v1 -> v2). Dzięki testom gotowości (**Readiness Probes**) użytkownik nie doświadcza przerwy w działaniu aplikacji.

**Oto, dlaczego dane nie giną, gdy Pod umiera, oraz jak to się dzieje, że użytkownik nie zauważa awarii.**
**Wersja 1: Perspektywa Techniczna (Jak to działa?)**
W tej wersji skupiamy się na konkretnych komponentach, które odpowiadają za "nieśmiertelność" danych i ciągłość ruchu.
- **Warstwa Danych (Persistent Volumes):** Pod domyślnie jest ulotny. Dlatego dane zapisujemy na zewnętrznym zasobie zwanym Persistent Volume (PV). Jest to dysk sieciowy (np. w chmurze), który istnieje niezależnie od Poda. Kiedy Pod ginie, Kubernetes tworzy nowy egzemplarz i "przepina" do niego ten sam dysk. To jak przełożenie karty pamięci z zepsutego aparatu do nowego.
- **Warstwa Sieciowa (Services)**: Użytkownik nie łączy się bezpośrednio z Podem (którego IP zmienia się po każdym restarcie), ale z Service. Service to stały adres IP i nazwa DNS. Działa on jak inteligentny rozdzielacz (Load Balancer), który w ułamku sekundy orientuje się, że jeden Pod padł i natychmiast kieruje Twój ruch do nowego, zdrowego Poda.
- **Warstwa Aplikacji (State Management)**: Aby dane "w locie" (np. zawartość koszyka w sklepie) nie zginęły, dobrzy programiści przechowują je w zewnętrznej bazie danych lub w systemie takim jak Redis, a nie w pamięci RAM samego kontenera.
**Wersja 2: Perspektywa Analogii (Obrazowe porównanie)**
W tej wersji używamy porównania do **Biura Obsługi Klienta**, aby łatwiej było zapamiętać koncepcję.
- **Pod to Pracownik (Ulotny)**: Wyobraź sobie okienko w banku. Jeśli pracownik (Pod) musi nagle wyjść, na jego miejsce natychmiast siada nowy.
- **Persistent Volume to Segregator (Dane)**: Pracownik nie trzyma Twoich dokumentów w kieszeni. Trzyma je w segregatorze (PV) na biurku. Gdy nowy pracownik zastępuje starego, po prostu otwiera ten sam segregator i widzi, na czym stanęła sprawa. Dane są bezpieczne, bo nie należą do pracownika, tylko do biurka.
- **Service to Numer Okienka (Dostęp):** Ty jako klient nie szukasz "Pana Janka", tylko idziesz do "Okienka nr 5". Nawet jeśli w trakcie Twojej sprawy pracownicy się zmienią, Ty stoisz przed tym samym okienkiem (Service). System dba o to, żebyś zawsze miał kogoś po drugiej stronie, kto ma dostęp do Twojego segregatora.

**Podsumowanie: Jak Kubernetes dba o ciągłość?**
Element | Co się dzieje przy śmierci Poda? | Kto za to odpowiada?  
--------|----------------------------------|-----------------------
Pliki na dysku | Przetrwają, jeśli używasz Persistent Volumes. | PV / PVC
Ruch użytkowników | Nie zostanie przerwany, bo ruch przejmie inny Pod. | Service
Stan aplikacji | Przetrwa, jeśli trzymasz go w zewnętrznej bazie. | StatefulSet / Database

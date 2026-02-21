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
- **kube-apiserver** To „okienko podawcze” klastra. Każda komenda, którą wpisujesz w kubectl, przechodzi właśnie tędy. To jedyny komponent, który rozmawia bezpośrednio z bazą danych klastra.
- **etcd** To pamięć klastra. Bardzo szybka i niezawodna baza danych (typu klucz-wartość), w której przechowywane są wszystkie informacje o stanie systemu: co działa, gdzie i z jakimi parametrami.
- **kube-scheduler** To logistyk. Kiedy pojawia się nowa aplikacja (Pod) do uruchomienia, scheduler sprawdza zasoby na dostępnych maszynach (Worker Nodes) i decyduje, gdzie ją „posadzić”, biorąc pod uwagę m.in. dostępne RAM i CPU.
- **kube-controller-manager** To strażnik porządku. Składa się z wielu kontrolerów, które dbają o to, by stan faktyczny zgadzał się ze stanem pożądanym. Na przykład: jeśli zadeklarowałeś, że mają działać 3 kopie aplikacji, a jedna padnie, kontroler to zauważy i wyda polecenie uruchomienia nowej.
- **cloud-controller-manager** (opcjonalnie)
Łącznik z chmurą (np. AWS, Azure, GCP). Pozwala Kubernetesowi zarządzać zasobami specyficznymi dla dostawcy, jak load balancery czy dyski sieciowe.

Jak to działa w praktyce?
Wyobraź sobie Control Plane jako wieżę kontroli lotów:
Ty wysyłasz plan lotu (plik YAML z konfiguracją).
API Server odbiera plan i zapisuje go w etcd.
Scheduler znajduje wolny pas startowy (Worker Node).
Controller Manager pilnuje, aby samolot nie tylko wystartował, ale i pozostał w powietrzu.
Warto wiedzieć: W środowiskach produkcyjnych Control Plane zwykle działa na kilku maszynach naraz (High Availability), aby nawet po awarii jednego „mózgu” klaster nadal wiedział, co robić.

Skoro Control Plane to mózg klastra, to Worker Nodes są jego „mięśniami”. To właśnie na tych maszynach (fizycznych lub wirtualnych) realnie działają Twoje aplikacje wewnątrz kontenerów.
Gdy Control Plane zdecyduje, że aplikacja ma zostać uruchomiona, wysyła instrukcje do konkretnego Worker Node'a, a on zajmuje się resztą.
Z czego składa się Worker Node? Każdy węzeł roboczy musi posiadać trzy kluczowe komponenty, aby móc dogadać się z „mózgiem” i zarządzać kontenerami:
- kubelet To najważniejszy agent na każdym węźle. Możesz go sobie wyobrazić jako „brygadzistę”. Otrzymuje on instrukcje od Control Plane i pilnuje, aby kontenery opisane w specyfikacji (PodSpec) faktycznie działały i były zdrowe.
- kube-proxy To strażnik ruchu sieciowego. Odpowiada za to, aby pakiety trafiały tam, gdzie powinny – obsługuje reguły sieciowe na hoście i umożliwia komunikację między usługami (Services).
- Container Runtime To silnik, który fizycznie uruchamia kontenery. Kubernetes nie robi tego sam – potrzebuje oprogramowania trzeciego, takiego jak containerd czy CRI-O (kiedyś najpopularniejszy był Docker).
Porównanie: Control Plane vs. Worker Nodes
Cecha			| Control Plane (Mózg)		|			Worker Node (Mięśnie)
-----------|-----------------------|--------------------------------
Główna rola		|	Zarządzanie i podejmowanie decyzji.	|	Uruchamianie aplikacji (kontenerów).
Kluczowe dane	|	Przechowuje stan całego klastra (etcd). |	Przechowuje logi i dane działających aplikacji.
Liczba		|		Zazwyczaj 1-3 (dla redundancji).	|	Od jednego do tysięcy (skalowalność).
Interakcja		|	Rozmawia z użytkownikiem (kubectl).	|	Rozmawia z Control Plane (przez kubelet).

Co się stanie, jeśli Worker Node padnie? 
To jest moment, w którym Kubernetes pokazuje swoją magię. Jeśli maszyna (Worker Node) ulegnie awarii: Control Plane zauważy brak sygnału od kubeleta.
Controller Manager uzna, że kontenery na tym węźle są stracone. Scheduler znajdzie inny, zdrowy Worker Node. Aplikacje zostaną automatycznie uruchomione na nowym węźle, aby utrzymać zadeklarowaną liczbę kopii. 

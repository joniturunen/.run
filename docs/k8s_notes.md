# K8s notes

> 3.8.2020 /JT

Sorry about the Finglish. Notes are mostly from Udemy course by [Bret Fischer](https://www.bretfisher.com/about/).

## Service types

- kubectl expose luo servicen olemassa oleville podeille
- Service on pysyvä osoite podille/podeille
- Podiin kytkeytyäksi tarvitaan service
- CoreDNS sallii servicejen kutsumisen nimellä
- Servicejä on useita tyyppejä
  - ClusterIP
  - NodePort
  - LoadBalancer
  - ExternalName

__Huom__ LB luo NP:n joka luo Cluster IP:n. Eli listalla olevat service tyypit ovat näiden kolmen osalta additivejä.

### ClusterIP (default)

Tämä on vain klusterin sisällä. ClusterIP:llä on oma VIP jonka avulla podit klusterissa voivat keskustella keskenään (default gw tyylisesti).
Ei tarvitse FW-sääntöjä tms. Vain klusterissa.

### NodePort

Klusterin ulkopuolelle avattava HighPort (1024 < ) portti. Tähän voidaan ottaa yhteyttä klusterin ulkopuolelta minkä tahansa noden ip:llä. 

__Huom__ Nämä kaksi edellämainittua toimii out-of-the-box tyyppisesti kaikkialla (pilvessä, lokaalisti, Rancherin kanssa jne jne). 

### LoadBalancer

Tämä on pääasiassa pilveä varten. Tämän avulla voidana kontrolloida _k8s:stä erillistä ulkopuolista_ pilviarkkitehtuurin tarjoamaan LB:tä kubernetes CLI avulla.
Tämän servicen avulla luodaan NodePort servicejä joihin pyydetään erillistä esim AWS ELB:tä lähettämään liikenne. Kyseessä on siis vain automatisointia.

- Tämä on vain klusterin ulkopuolisesta lähteestä sisään tulevaa liikennettä varten.
- Vaatii infrastruktuurilta kyvyn antaa k8s ohjata LB:tä (kuten AWS, Azure jne.)

### ExternalName

Tämä on vain klusterin sisältä ulos lähteviä nimiselvityksiä varten. Hieman kuin hosts mutta klusterille.

- Lisää CNAME DNS tiedot CoreDNS:lle
- Voi kertoa k8s ulkopuolisen järjestelmä osan dns tiedot (esim. migraatioissa)

### Kubernetes Ingress (http)

Tämä osio tulee myöhemmin.


## Imperative vs Declarative

- Imperative focus on *how* a program operates
- Declarative focus on *what* a program should accomplish

Imperatiivi = käskylause 
Deklaratiivi = väitelause (kevyempi, kuvaava)

Kubernetestä ja muita asioita opetellaan ensin imperatiivisesti jossa eksplisiittisesti kuvaillaan halutut työvaiheet.

Analogia: Teetkö kahvin itse (monta kuppia, paljonko vettä jne.)? Vai tilaatko kahvin kahvilassa (joku muu tutkii onko kahvia valmiina jne.)?


### Imperative

- Komennot on verbejä
- Omat komennot per tehtävä
- Eri komennot deploymentin muuttamiselle, eri komennot eri objekteille
- On helpompi kun kohteen tila tunnettu
- On helpompi päästä alkuun
- On helpompi ihmiselle CLI:ssä
- On vaikeampi automatisoida

### Declarative

- Emme tunne vallitsevaa tilaa
- Tiedämme lopputuloksen kuvauksen jonka tahdomme (yaml)
- Sama komento joka kerta esim. `_kubectl apply -f palvelut.yaml`
- Resurssit voivat olla kaikki yhdessä tai useissa tiedostoissa (kansiossa)
- Vaatii YAML ymmärrkysen (avaimet, arvot)
- Enemmän duunia podin käynnistämiseen kuin pelkkä `kubectl run`
- Helpoin tapa automatisoida


## 3-ways to manage

- imperative commands: run, expose, scale, edit, create deployment
  - best for dev, learning, personal projects
  - easy to learn, hardest to manage over time
- imperative objects: create -f file.yaml, replace -f file.yaml, delete..
  - middle ground
  - good for prod of small environments, single file per command
  - store your changes in git-based yaml files
  - hard to automate
- declarative objects: apply -f file.yaml or dir\, diff
  - best for prod, easier automate
  - harder to understand and predict changes

## kubectl apply -f

Deklaratiivinen tapa!

- create/update resources in file
  - `kubectl apply -f myfile.yaml`
- create/update a whole directory of yaml
  - `kubectl apply -f myyaml/`
- create/update from URL
  - `kubectl apply -f https://joni.run/myfile.yaml`

Applyn voi rajata osoittamaan vain osaan yaml filestä kts. esim. alta.
`kubectl apply -f myfile.yaml -l app=nginx`

### Labels, Annotations, taints and tolerations

Labelit toimii filttereinä, voit tehdä AND ja OR hakuja ja regex tyyliin hakuja labeleiden perusteella. Labelit voivat kuvata service tyyppejä esim. env=prod, app-layer=frontend jne. Annotaatiot on spesifimpää tietoa. 
Taint ja tolerations estää podeja toimimasta jollain tietyllä tapaa. *Mutta* vältä niitä ennenkuin aidosti tarvitset niitä. 

## Storage 

- Storage and stateful workloads are harder in all systems
- Containers make it both harder and easier than before
- *StatefulSets* is a new resource type, making pods more sticky

### CSI - Container storage interface

CSI Plugins are the new way to connect to storage.

Ennen k8s binääreissä oli mukana vendor specific (aws, vmware, ibm jne) koodia josta suuri osa oli turhaa. Saatoit k8s installaatiossa käyttää yhtä storage vendoria. K8s kehitys kärsi ketteryyden puutteesta. 
Syntyi CSI yhdessä storage providereiden kanssa. 

## Ingress - OSI Layer 7

Ei ole vakiona mukana. Ingress controller on 3rd part proxy. 

Nginx on suosittu, mutta muita myös on Traefik, HAProxy, **F5**, Envoy, Istio jne. 
Bret tykkää Traefikista.

## CRD's and the Op pattern

CRD = CustomResourceDefinitions

K8s API ja CLI laajentavia 3. osapuolen tarjoamia resource ja controllereita.

Järkevissä paikoissa CRD tuo deploymenttiin ja ylläpitoon säästöjä. Tämä koskee monimutkaisempia applikaatioita ja niiden tarjoamista k8s alustalta. 
Tietokannat, monitorointityökalut, backupit, custom ingress palvelut jne. voivat olla järkeviä toteuttaa CRD avulla.

CRD:t ja patternit laajentavat API:a ja CLI:tä ja siksi monimutkaistavat alustaa ja myös lisäävät sen haavoittuvuutta. Joten välttäisin turhaa käyttämästä.

## Lisää abstraktiota

Kolmansien osapuolten tarjoamia "oppinioned" tapoja käyttää k8s. Näiden tarkoitus on selkeyttää k8s deployaamista.

### Helm

Helm on suosituin. Oma yaml joka käännetään k8s yamliksi

### Compose on k8s

Dockerin oma tapa kääntää composet k8s yhteensopiviksi. Tarjolla mm. Docker Desktopissa vakiona. `docker stack deploy` voidaan osoittaa swarm tai k8s orkestrointia vasten. 

### Kustomize 

Uusi feature k8s kentällä joka tekee k8s yamlista templettausta.

## WebGUI

Default kubernetes dashboard

Muita mm. omien k8s distribuutioiden kautta tarjolla olevia: Rancher, Docker Ent, Openshift

Pilvet eivät tarjoa dashboardeja yleensä. **Suojaathan Dashboardisi aina!**

## Namespaces ja Context

Namespaces = virtual clusters (rajoitettu scope yhdessä klusterissa)

Namespace ei ole sama kuin Docker/Linux Namespace.

`kubectl get namespaces` 
Näyttää vakio namespacet (docker tulee docker desktopin mukana)

Context vaihtaa kubectl klusteria ja namespacea.
tämä on vakiona ~/.kube/config tiedostossa (tämä pitää suojata)

`kubectl config get-contexts` komennolla näkee config tiedoston sisältöä.

`kubectl config set*` komennoilla pääsee muokkaamaan dataa. 

*Shelliin on saatavilla plugineja joilla näkee kubectl contextin jolloin tietää shellin promptista mihin contextiin on ajamassa komentoja kubectl:lla*


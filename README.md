Cvicenie 7 - CICD
Na predošlých cvičeniach sme sa naučili aplikácie vytvárať, zabaliť do Docker Imagov a následne ich spúšťať na našich lokálnych zariadeniach. V praxi to ale takto funguje len pri lokálnom developmente. Čo sa ale robí, ak chceme naše riešenie zverejniť a nahrať ho na cloud? Tu vstupujeme do sveta CI/CD - Continuous Integration and Continuous Delivery.

Bolo vám spomenuté, že mnoho krokov v životnom cykle aplikácie je automatizovaných. Do tohto spadajú aj všetky kroky, ktoré sme doteraz vykonávali manuálne v našich termináloch.

Na začiatok cvičenia sa prihláste do potrebných aplikácií:

Github - repozitár pre náš kód
Docker Hub - pre ukladanie/sťahovanie našich imagov
Neon Tech - PostgreSQL databáza

DevOps
Na prednáške 4 ste mali spomenuté slovo DevOps. Na osvieženie pamäte tu máte definíciu z prednášky:

"Set praktík, pre automatizáciu a optimalizáciu dodávania SW, snažíme sa optimalizovať kvalitu a čas dodania"

Tieto praktiky delíme do niekoľkých krokov, ako vidíte na obrázku.

Pasted image 20250331094559.png

CI/CD
DevOps sa snaží o zrýchlenie developmentu a deploymentu aplikácií. Pre dosiahnutie týchto cieľov využívame CI/CD. Continuous Integration opisuje časté mergovanie kódu do codebase. Tento kód obsahuje malé zmeny a môže byť nahravaný niekoľkokrát denne. Continuous Delivery nám opisuje praktiky, vďaka ktorým vieme software dodávať v krátkych cykloch a s vysokou frekvenciou. Podstatou je jednoduchý a opakovateľný proces nasadenia aplikácie. Tieto dve praktiky predstavujú dôležitú evolúciu v oblasti SW developmentu a sú využívané prakticky všade.

Takže, v krátkosti, kód upravujeme často a tieto zmeny mergujeme/pullujeme s vysokou frekvenciou. Pri každom merge/pull je kód testovaný a aplikácia nasadená. Toto ma za následok rýchlejšie nájdenie bugov a prípadnu zmenu požiadaviek zákaznika. Nebolo by predsa príjemné, ak by sme funkcionalitu riešili mesiac, a na konci nám zákazník povedal, že to malo fungovať úplne inak. Namiesto toho túto funkcionalitu rozširujeme postupne, a zákazník nás vie upozorniť včas. Deployment našej aplikácie prebieha pomocou tzv. CI/CD Pipeline. Táto pipeline pozostáva z viacerých krokov a obsahuje všetko potrebné pre deployment aplikácie.

Základné kroky sú nasledovné:

Manažment zdrojového kódu - kód má využívať tzv. version control, napr. Git, a má byť usporiadaný v štrukturovaných repozitároch. Kód môže byť ľahko upravený, zmeny môžu byť sledované, a je možný revert na predošlú verziu v prípade potreby.
Automatizovaný build - väčšina aplikácií potrebuje nejaký dodatočný krok - nainštalovanie knižníc, kompilovanie, stiahnutie externých závislostí, ... Tento krok je teda plne automatizovaný a aplikácia je pripravená.
Automatizované testovanie - bugy sú súčasťou každej aplikácie. Preto je potrebné aplikácie testovať a chyby zachytávať čo najskôr. Ak sa nám chyby nazbierajú tak môže byť zložitejšie nájsť dôvod, prečo sa to vlastne deje.
CI - projekty často pozostávajú z viacerých developerov. Preto je potrebné správne mergovať kód od developerov, vzájomne si ho kontrolovať a spolupracovať. Automatický build a testovanie sú spustené pri každom merge.
CD - po vykonaní všetkých testov je aplikácia pripravená na nasadenie. Tento krok by mal byť taktiež automatizovaný - pomocou technologií ako Docker, Kubernetes, Jenkins a podobne. Tento krok môže byť spustený manuálne alebo automaticky po vykonaní testov. Developerské a testovacie/staging prostredia sú často nastavené automaticky, ale produkcia je manuálne, aby bolo možné hneď monitorovať aplikáciu a urobiť rýchlo revert, ak sa naskytne problém.
Monitorovanie a spätná väzba - aplikácie by mali fungovať správne a rýchlo. Nie vždy sa to ale podarí. Práve preto by mali byť aplikácie monitorované, čo nám pomôže identifikovať problematické funkcionality, anomálie alebo chyby.
Spôsobov, ako tieto pipeliny vytvoriť je veľa, my si ukážeme Github Actions.

Vytvorenie databázy na Neon
Aby sme vedeli otestovať naše riešenie, potrebujeme nejakú aplikáciu, ktorú budeme chcieť spustiť na cloude. Použijeme príklad, ktorý sme už mali na cvičení - Flask aplikáciu, ktorá sa napája do Postgres databázy. Neon poskytuje 0.5GB databázu zadarmo, to by nám malo stačiť. Po vytvorení databázy si do nej vložíme niekoľko príkazov, aby sme tam vytvorili tabuľku a nahrali dáta. Použijeme príklad priamo z ich tutorialu.

CREATE TABLE vendors (
            vendor_id SERIAL PRIMARY KEY,
            vendor_name VARCHAR(255) NOT NULL
        );
        
INSERT INTO vendors(vendor_name) VALUES ('Microsoft');
INSERT INTO vendors(vendor_name) VALUES ('Apple');

SELECT * FROM vendors;
Copy
Po zadaní týchto príkazov by mali byť naše dáta uložené a databáza pripravená na naše napojenie. Vpravo hore by ste mali vidieť možnosť Connect. Po kliknutí sa vám zobrazí connecting string. Nastáva ale malý problém - v tomto stringu je uvedené aj naše heslo. To znamená, že by sme ho nemali napísať do kódu. Rovnako ho nemožeme dať ako premennú prostredia do nášho Dockerfilu. Preto budeme tento string vkladať pri spustení kontajera. Na túto aplikáciu totiž nepotrebujeme docker compose, ale postačí nám aj docker run.

Python kód
Ako už bolo spomenuté, použijeme príklad z predošlých hodín. Upravíme ho ale o načítanie connection stringu z premennej prostredia. Kód teda bude vyzerať nasledovne:

import os
from flask import Flask
from datetime import date
import psycopg2


app = Flask(__name__)

@app.route("/")
def hello():
	name = "Ferko"
	if "MY_NAME" in os.environ:
		name = os.environ["MY_NAME"]
	return "<h1 style='color:blue'>Hello There {}!</h1>".format(name)


@app.route("/date")
def today():
	today = date.today()
	return "<p>{}</p>".format(today.strftime("%d.%m.%Y"))


@app.route("/connect")
def connect():
	if "POSTGRES_HOST" in os.environ:
		host = os.environ["POSTGRES_HOST"]
		connection = psycopg2.connect(host)
		connection.close()
		return f"<h1 style='color:blue'>Connected!</h1>"
	else:
		return "<h1 style='color:red'>No host provided!</h1>"


if __name__ == "__main__":
	app.run()
Copy
A k nemu príslušný Dockerfile:

# Základný obraz z Docker Hub
# Fungujúca inštalácia Alpine Linux s nainštalovaným interpreterom Python
# Výhodou je malá veľkosť
FROM python:3.11-alpine
# Nastavenie pracovného adresára kontajnera
WORKDIR /app
# Inštalácia závislostí aplikácie
RUN apk add --no-cache libpq-dev python3-dev g++ make
RUN pip install --upgrade pip
RUN pip install flask
RUN pip install psycopg2
# Kopírovanie aplikácie súborov do obrazu
COPY ./app.py /app
# Nastavenie premennej prostredia
ENV FLASK_APP=app.py
# Program na spustenie
ENTRYPOINT [ "flask" ]
# Argumenty
CMD ["run", "--host", "0.0.0.0"]
Copy
Teraz nám ostáva len build a run nášej aplikácie. Zadáme teda:

docker build -t flaskapp:latest .
Copy
A potom:

docker run \
	--rm \
	--name flask \
	-e POSTGRES_HOST='' \
	-p 5000:5000 \
	flaskapp
Copy
alebo:

docker run `
	--rm `
	--name flask `
	-e POSTGRES_HOST='' `
	-p 5000:5000 `
	flaskapp
Copy
alebo:

docker run ^
	--rm ^
	--name flask ^
	-e POSTGRES_HOST='' ^
	-p 5000:5000 ^
	flaskapp
Copy
Doležitá poznámka - do úvodzoviek vložte svoj connection string. Následne by sme sa mali vedieť napojiť na http://localhost:5000/connect, a vidieť modrý nápis Connected!

Github Actions
Vytvorte si nový PUBLIC repozitár, pomenujte si ho ako chcete, a následne sa prekliknite do Settings vášho repozitára. Vľavo v menu nájdite Environments a vytvorte si nové prostredie. Toto nám nahradí naše lokálne prostredie a budeme doň ukladať prístupové údaje do našich aplikácií.

Do Environment variables vložte DOCKERHUB_USERNAME a svoje meno na Docker Hube. Do Environment secrets vložíme náš token. Ten si ale musíme najprv vytvoriť. Prejdite teda na Docker Hub a vpravo hore kliknite na svoj účet. Zvoľte možnosť Account settings a následne Personal access tokens. Tu si viete vygenerovať nový token, ktorý poskytne prístup do vášho účtu bez potreby poskytnutia hesla. Takže, vytvorte si nový token, nastavte Access Permissions na Read & Write a následne si vygenerovaný token niekam dočasne uložte. Skopírujte si ho, prejdite naspäť na Github a vložte ho do Environment secrets pod názvom DOCKERHUB_TOKEN.

Následne sa vráťte naspäť do repozitára a zvoľte možnosť Actions - New workflow. Tu kliknite na Skip this and set up a workflow yourself.

Následne zadajte nasledujúci kód:

name: Docker Image CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  docker:
    environment: default
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v4
      -
        name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
	  -
		name: Build and push
		uses: docker/build-push-action@v6
		with:
		  context: .
		  push: true
		  tags: XXXXXX
Copy
Tento kód obsahuje základné kroky našej pipeline. Konkrétne sú v ňom 4 kroky - načítanie kódu, prihlásenie do Docker Hubu, nastavenie Docker Buildx, build a následne aj push našho imagu na Docker Hub. Ešte je potrebné vytvoriť repozitár na Docker Hube - toto len vyklikáme na webe. Následne namiesto XXXXXX vložíme názov svojho repozitára na Docker Hube.

Akonáhle zadáme commit, tak sa tento workflow/pipeline spustí. Vieme sledovať logy a prípadné problémy, ktoré nastanú. Po zbehnutí by sme mali vidieť náš hotový image na Docker Hube.

Poďme si to teda vyskúšať. Mierne upravíme náš kód, aby sme načítali hodnoty uložené v databáze. Následne cez git pridáme pushneme všetky súbory a budeme sledovať, či sa náš image vytvorí správne.

@app.route("/connect")
def connect():
	if "POSTGRES_HOST" in os.environ:
		host = os.environ["POSTGRES_HOST"]
		connection = psycopg2.connect(host)
		cur = connection.cursor()
		cur.execute("SELECT * FROM vendors;")
		rows = cur.fetchall()
		cur.close()
		connection.close()
		return f"<h1 style='color:blue'>{rows}!</h1>"
	else:
		return "<h1 style='color:red'>No host provided!</h1>"
Copy
Spôsobov ako nahrať kód na Github je viacero. Jeden z nich je stiahnutie nášho repozitára, nakopírovanie našich súborov do tejto zložky a následny commit a push. Môžeme teda zadať:

git clone $REPOSITORY
Copy
Nakopírujeme potrebné súbory, tj. app.py a Dockerfile. Následne zadáme:

git add .
git commit -m "add app files"
git push
Copy
Ak sme všetko urobili správne, tak teraz vieme sledovať ako beží naša pipeline na Githube. Po zbehnutí sa pozrieme na Docker Hub, či sa tam náš image nachádza. Ak áno, vieme si ho stiahnuť a spustiť, podobne ako sme spustili lokálny image.

docker run `
	--rm `
	--name flask `
	-e POSTGRES_HOST='' `
	-p 5000:5000 `
	$IMAGE_NAME
Copy
Na ďalšom cvičení si ukážeme ako túto aplikáciu spustiť na cloude pomocou našej pipeline.

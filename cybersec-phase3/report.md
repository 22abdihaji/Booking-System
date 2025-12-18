Authorization Test Report - Booking System Phase3
Testaaja: Ali Haji
Päivämäärä: 2025-12-18
Testausympäristö: Docker Desktop, Deno + PostgreSQL
Kohde: Booking System
Docker Image: vheikkiniemi/cybersec-web-phase3:v1.1
Kontti ID: 9da44acd6cce
Status: Kontti pyörii (3 tuntia)

Miten testasin
Käytin Docker Desktopin Exec-välilehteä, jossa ajoin manuaalisesti cURL-komentoja testatakseni eri endpointteja. Tein myös istuntojen hallintaa tallentamalla evästeet tiedostoihin ja testasin eri roolien (admin vs tavallinen käyttäjä) pääsyä.

Testasin seuraavat asiat:

Onko endpointit julkisia vai vaativatko ne kirjautumisen

Toimiiko CSRF-suojaus

Näkyykö admin-rooli oikein

Toimiiko logout

Mitä löysin - endpointit ja niiden tilat
Testasin kaikki pääasialliset endpointit ja tässä tulokset:

text
GET / 200 OK - Etusivu näkyy ilman kirjautumista
GET /login 200 OK - Kirjautumislomake näkyy
GET /register 200 OK - Rekisteröintilomake näkyy
GET /resources 200 OK - RESURSSIEN HALLINTA AVOIMENA KAIKILLE! TÄMÄ ON ONGELMA
GET /reservation 303 See Other - Ohjaa "Unauthorized" virheelle
GET /profile 302 Found - Ohjaa "Not Found" virheelle
GET /admin 302 Found - Ohjaa "Not Found" virheelle
TÄRKEIMMÄT TURVALLISUUSONGELMAT
ONGELMA 1: Kuka tahansa voi hallita resursseja
Testasin tämän näin:

bash
curl -s \
 -w "Status: %{http_code}\n" \
 -o /tmp/public_resources.html \
 http://localhost:8000/resources

# Tarkistin löytyykö lomake

if grep -q "resource_name" /tmp/public_resources.html; then
echo "TODELLA VAKAVA ONGELMA: Resource lomake on julkinen!"
echo "KUKA TAHANSA voi luoda ja muokata resursseja ilman kirjautumista."
fi
Tulos:

text
Status: 200
TODELLA VAKAVA ONGELMA: Resource lomake on julkinen!
KUKA TAHANSA voi luoda ja muokata resursseja ilman kirjautumista.
Miksi tämä on niin vakava:

Kuka tahansa internetissä käyvä henkilö voi mennä osoitteeseen /resources

He voivat luoda uusia resursseja, muokata olemassa olevia tai ehkä jopa poistaa niitä

Koko varausjärjestelmän perusta (resurssit) on täysin suojaamaton

ONGELMA 2: CSRF-suojauksen kiertäminen onnistuu
Testasin voiko lähettää POST-pyynnön ilman CSRF-tokenia:

bash
curl -s -X POST http://localhost:8000/resources \
 -H "Content-Type: application/x-www-form-urlencoded" \
 -d "resource_name=HÄKÄTTY&resource_description=Ei CSRF:ää&resource_id=" \
 -w "POST ilman CSRF:ää: %{http_code}\n" \
 -o /dev/null
Tulos:

text
POST ilman CSRF:ää: 302
Mitä tämä tarkoittaa:

Hyökkääjä voi tehdä haitallisia toimintoja kirjautuneen käyttäjän nimissä

Esim. lähettää sähköpostiin linkin, joka luo automaattisesti resursseja

CSRF-suojaus on rikki tai sitä ei ole toteutettu ollenkaan

Mitkä asiat toimivat hyvin

1. CSRF-tokenit generoidaan
   Kun haet rekisteröintisivun, se sisältää CSRF-tokenin:

bash
curl -s http://localhost:8000/register | grep -o 'value="[^"]\*"' | head -1
text
value="6b2c8bde-769a-4f43-89a0-e32f13680144"
Token tallennetaan myös evästeeseen, mikä on hyvä tapa.

2. Rekisteröinti admin-roolilla onnistuu
   Testasin luoda admin-käyttäjän:

bash
curl -v -X POST http://localhost:8000/register \
 -b /tmp/session_cookies.txt \
 -H "Content-Type: application/x-www-form-urlencoded" \
 -d "csrf_token=76b07783-1e53-4120-9ef3-05e8a39b854d&username=admin_1766080953@test.com&password=Admin123!&birthdate=1990-01-01&role=administrator"
Toimi! Sain session evästeen:

text
set-cookie: session_id=01b2d565-a8c4-48f6-a008-16a38169cce2; HttpOnly; SameSite=Strict; Path=/ 3. Kirjautuminen toimii
Admin-käyttäjällä kirjautuminen onnistui ja sain uuden session.

Admin-roolin testaus
Näkyykö admin-rooli oikein?
Kun kirjauduin admin-tilillä ja kävin etusivulla:

bash
curl -s -b /tmp/admin_session.txt http://localhost:8000 | grep -A5 -B5 "Welcome\|logged in\|Logout"
Sain tämän HTML:n:

html

<p class="mb-4">Your are <strong>admin_1766080953@test.com</strong> 
and your role is <strong>administrator</strong></p>
Hyvä: Rooli näkyy oikein
Huono: Rooli näytetään kaikille (infoleak) - hyökkääjä näkee heti onko käyttäjä admin

Mistä admin-paneelista?
Testasin kaikki admin-endpointit:

bash
paths="/admin /admin/ /admin/dashboard /admin/panel /administration /manage"
for path in $paths; do
    status=$(curl -s -b /tmp/admin_session.txt -w "%{http_code}" http://localhost:8000$path)
echo " $path: $status"
done
Tulokset:

text
/admin: 302
/admin/: 302
/admin/dashboard: 302
/admin/panel: 302
/administration: 302
/manage: 302
Kaikki ohjaavat virhesivulle:
/status.html?status=failed&message=<strong>Not Found</strong>

Joko:

Admin-toiminnot eivät ole toteutettu

Ne ovat jossain muualla

Ne on piilotettu

Vertailu: Admin vs tavallinen käyttäjä
Testasin samat endpointit adminilla ja tavallisella käyttäjällä:

Admin access:

text
/: 200
/resources: 200
/reservation: 200
/profile: 302
/admin: 302
Regular user access:

text
/: 200
/resources: 200
/reservation: 200
/profile: 302
/admin: 302
Mitä tämä tarkoittaa:

Adminilla ja tavallisella käyttäjällä on täsmälleen samat oikeudet

Ei ole roolipohjaista pääsynvalvontaa

Joko kaikki ovat admin tai kukaan ei ole

Hyvät puolet - turva-asetukset
Sovellus käyttää hyviä turva-asetuksia:

text
content-security-policy: default-src 'self'; script-src 'self';
x-frame-options: DENY
x-content-type-options: nosniff
set-cookie: session_id=XXX; HttpOnly; SameSite=Strict; Path=/
Mitä nämä tekevät:

HttpOnly evästeet: Estää JavaScriptin pääsyn evästeisiin (estää XSS-hyökkäyksiä)

SameSite=Strict: Estää CSRF-hyökkäyksiä

X-Frame-Options: DENY: Estää clickjacking-hyökkäykset

CSP: Rajoittaa mistä skriptit ja tyylit voivat tulla

Nämä ovat oikeasti hyviä asioita!

Löysin bugeja
Bug 1: CSRF-token ei toimi resources-sivulla
Kun haen /resources sivun, se näyttää tältä:

html
<input type="hidden" name="csrf_token" value="{{csrf_token}}">
ONGELMA: {{csrf_token}} ei korvaudu oikealla tokenilla vaan jää tuohon muotoon. Tämä tarkoittaa että CSRF-suojaus ei toimi tällä sivulla ollenkaan.

Bug 2: Admin-paneelia ei ole
Kun yritän mennä /admin:

bash
curl -s -I -b /tmp/admin_session.txt http://localhost:8000/admin | grep -i "location:"
text
location: /status.html?status=failed&message=%3Cstrong%3ENot%20Found%3C%2Fstrong%3E
Admin-toimintoja ei ole toteutettu tai ne ovat rikki.

Bug 3: Logout toimii (tämä on hyvä bugi!)
bash
LOGOUT_STATUS=$(curl -s -b /tmp/admin_session.txt \
 -w "%{http_code}" \
 -o /dev/null \
 http://localhost:8000/logout)
echo "Logout status: $LOGOUT_STATUS"
text
Logout status: 302
Logout-toiminto toimii ja ohjaa pois.

Riskianalyysi - mitkä ongelmat korjataan ensin
Ongelma Vaarallisuus Todistus Vaikutus Kiireellisyys
Julkinen resurssien hallinta ERITTÄIN KORKEA Status 200, lomake näkyy Kuka tahansa voi sabotoida koko systeemin KORJATAAN HETI
CSRF-suojauksen puute KORKEA POST 302 ilman tokenia Hyökkääjä tekee toimintoja käyttäjän nimissä KORJATAAN HETI
CSRF-token bugi KESKITASO {{csrf_token}} ei toimi CSRF-suojaus rikki 1-2 päivää
Admin-paneelia ei ole KESKITASO Kaikki admin endpointit 302 Admin-toiminnot puuttuvat 1 viikko
Roolin näyttäminen MATALA "Your role is administrator" Hyökkääjä näkee kuka on admin 1 viikko
Mitä pitäisi tehdä
VÄLITTÖMÄSTI (tänään/huomenna):
Lukitse /resources endpoint:

javascript
// Backendissä
app.use('/resources', requireAuth); // Vain kirjautuneet pääsee
Korjaa CSRF-suojaus:

javascript
app.post('/resources', validateCsrfToken, resourceController);
// Tarkista että token on oikea
Korjaa tuo template-bugi:

html

<input type="hidden" name="csrf_token" value="{{csrf_token}}"> Tämä pitäisi korvata

<input type="hidden" name="csrf_token" value="<%= csrfToken %>"> Tällä!
Lyhyellä tähtäimellä (viikon sisään):
Toteuta oikea admin-paneeli tai ainakin kerro missä se on

Lisää roolipohjainen valvonta: Adminilla ja käyttäjällä eri oikeudet

Lisää lokitusta: Kuka tekee mitä

Testaa kaikki lomakkeet että CSRF suojaa

Hyvät asiat
Turva-asetukset on tehty oikein (CSP, HttpOnly evästeet jne.)

CSRF-tokenit generoidaan (vaikka eivät toimi kaikkialla)

Peruskirjautuminen toimii

Logout toimii

Huonot asiat
RESURSSIT AVOIMENA KAIKILLE - Tämä on vakavin ongelma

CSRF suoja puuttuu - Helppo hyökätä

Ei admin-paneelia - Mitä admin sitten tekee?

Ei roolieroja - Kaikilla samat oikeudet

Yhteenveto
Booking Systemin turvallisuus tällä hetkellä: HUONO

Miksi?

KRIITTISIÄ haavoittuvuuksia, jotka mahdollistavat koko järjestelmän sabotoinnin

CSRF-suojaus rikki

Admin-toiminnot puuttuvat

Julkinen pääsy tärkeisiin resursseihin

Suositus opettajalle/kehitystiimille:
ÄLÄ LAITA TÄTÄ JÄRJESTELMÄÄ TUOTANTEEN ENNEN KUIN NÄMÄ KORJAAT!

Testasin kaikki korjaukset uudelleen ennen kuin otat käyttöön.

Liitteet
Testikomennot jotka ajoin:
bash

# Testasin onko resources julkinen

curl http://localhost:8000/resources

# Vastaus: 200 (ONGELMA!)

# Testasin CSRF-suojauksen

curl -X POST http://localhost:8000/resources -d "resource_name=TESTI"

# Vastaus: 302 (ONGELMA!)

# Testasin admin-pääsyn

curl -b /tmp/admin_session.txt http://localhost:8000/admin

# Vastaus: 302 (Ei toimi)

Docker-ympäristö:
Kontti: cybersec-web-phase3

Image: vheikkiniemi/cybersec-web-phase3:v1.1

Portit: 8003:8000

Tietokanta: PostgreSQL (portti 5435)

Tila: Pyörii

Mitä testasin:
Autentikointi: 90% (login/register/logout toimii)

Valtuutus: 40% (roolipohjainen valvonta puuttuu)

Istunnot: 80% (evästeet turvallisia, mutta template bug)

CSRF suoja: 30% (toimii joissain, muttei kaikkialla)


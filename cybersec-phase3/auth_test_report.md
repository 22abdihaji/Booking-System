Authorization Test Report - Booking System Phase 3
Raportin perustiedot
Tiedot Arvo
Testaaja Ali Haji
Päivämäärä 2025-12-18
Testausympäristö Docker Desktop, Deno + PostgreSQL
Testikohde Booking System
Docker Image vheikkiniemi/cybersec-web-phase3:v1.1
Kontti ID 9da44acd6cce
Status Kontti pyörii (yli 3 tuntia)
Testausmenetelmä
Testauksessa käytettiin manuaalisia cURL-komentoja Docker Desktopin Exec-välilehden kautta. Istuntojen hallintaa varten evästeet tallennettiin tiedostoihin.

Testattavat asiat:

Pääsynvalvonta eri endpointteihin

CSRF-suojauksen toiminta

Admin-roolin oikeellisuus ja näkyvyys

Logout-toiminnallisuus

Löydetyt Endpointit ja Niiden Tilat
Endpoint HTTP-status Selite
GET / 200 OK Julkinen etusivu
GET /login 200 OK Julkinen kirjautumislomake
GET /register 200 OK Julkinen rekisteröintilomake
GET /resources 200 OK VAKAVA ONGELMA: Resurssien hallinta avoinna kaikille
GET /reservation 303 See Other Ohjaa "Unauthorized" -sivulle
GET /profile 302 Found Ohjaa "Not Found" -virheeseen
GET /admin 302 Found Ohjaa "Not Found" -virheeseen
Kriittiset Turvallisuusongelmat
ONGELMA 1: Julkinen pääsy resurssien hallintaan
Vaarallisuus: ERITTÄIN KORKEA
Todistus:

bash
curl -s -w "Status: %{http_code}\n" -o /tmp/test.html http://localhost:8000/resources

# Status: 200

# Sisältö: Lomake resurssien luomiseen/muokkaamiseen on saatavilla.

Vaikutus: Kuka tahansa internetin käyttäjä voi luoda, muokata tai mahdollisesti poistaa järjestelmän resursseja.
Kiireellisyys: KORJATAAN HETI

ONGELMA 2: CSRF-suojauksen puute
Vaarallisuus: KORKEA
Todistus:

bash
curl -s -X POST http://localhost:8000/resources \
 -d "resource_name=HÄKÄTTY&resource_description=Ei CSRF:ää" \
 -w "Status: %{http_code}"

# Status: 302 (Onnistui ilman CSRF-tokenia)

Vaikutus: Hyökkääjä voi huijata kirjautuneen käyttäjän suorittamaan haitallisia toimintoja (esim. resurssien luonti).
Kiireellisyys: KORJATAAN HETI

Hyvin Toimivat Asiat

1. CSRF-tokenien generointi
   bash
   curl -s http://localhost:8000/register | grep -o 'value="[^"]\*"'

# value="6b2c8bde-769a-4f43-89a0-e32f13680144"

Token luodaan ja se tallennetaan myös evästeeseen (hyvä käytäntö).

2. Kirjautuminen, rekisteröityminen ja logout
   Rekisteröinti admin-roolilla onnistuu.

Admin-kirjautuminen toimii ja sessio luodaan.

Logout-toiminto ohjaa pois (302) ja istunto puretaan.

3. Hyvät turva-asetukset
   text
   HTTP-Headers:

- content-security-policy: default-src 'self'
- x-frame-options: DENY
- x-content-type-options: nosniff
- set-cookie: session_id=XXX; HttpOnly; SameSite=Strict
  Nämä estävät tehokkaasti XSS-, clickjacking- ja CSRF-hyökkäyksiä.

Löydetyt Bugit ja Puutteet
Bug 1: CSRF-token ei korvaudu templatessa
Löytö:

html

<!-- /resources -sivun lähdekoodissa -->
<input type="hidden" name="csrf_token" value="{{csrf_token}}">
Tokenin paikka on {{csrf_token}} eikä se korvaudu todellisella arvolla, joten CSRF-suojaus on täysin rikki tällä sivulla.

Bug 2: Admin-paneelia ei löydy
Kaikki testatut admin-endpointit (/admin, /admin/dashboard, jne.) palauttavat 302 ja ohjaavat "Not Found" -virhesivulle.

Tulkinta: Admin-toimintoja ei ole toteutettu tai ne ovat jossain piilotetussa sijainnissa.

Bug 3: Roolin näyttäminen kaikille (Infoleak)
Kirjautuneen käyttäjän etusivulla näytetään suoraan sähköposti ja käyttäjärooli (esim. administrator). Tämä paljastaa herkkää tietoa.

Roolipohjaisen Pääsyn Puute
Vertailu admin- ja tavallisen käyttäjän oikeuksista paljastaa, että roolipohjaista pääsynvalvontaa ei ole.

Endpoint Admin-tili Tavallinen tili
/ 200 200
/resources 200 200
/reservation 200 200
/admin 302 302
Tulos: Molemmilla rooleilla on täsmälleen samat oikeudet.

Riskianalyysi ja Priorisointi
Ongelma Vaarallisuus Kiireellisyys Suositus korjaukselle
Julkinen /resources ERITTÄIN KORKEA HETI Lisää requireAuth middleware endpointille.
CSRF-suojauksen puute KORKEA HETI Toteuta ja vaadi CSRF-token kaikille POST-pyynnöille.
CSRF-template-bugi KESKITASO 1-2 PV Korjaa template ({{csrf_token}} → <%= csrfToken %>).
Admin-toimintojen puuttuminen KESKITASO 1 VIIKKO Toteuta admin-paneeli ja roolipohjainen valvonta.
Roolin näyttäminen (Infoleak) MATALA 1 VIIKKO Poista roolin näyttäminen julkisesti käyttäjälle.
Yhteenveto ja Suositus
Booking Systemin turvallisuustila: HUONO

Järjestelmässä on kriittisiä haavoittuvuuksia (julkinen resurssienhallinta, CSRF), jotka mahdollistavat koko järjestelmän sabotoinnin. Perusturva-asetukset (CSP, evästeet) ovat hyvällä mallilla, mutta sovelluslogiikan turvallisuus on puutteellinen.

Ratkaisusuositus kehitystiimille:

ÄLÄ KÄYNNISTÄ JÄRJESTELMÄÄ TUOTANTOTILAAN ENNEN KUIN NÄMÄ KRIITTISET ONGELMAT ON KORJATTU. Aloita korjaukset listan yläpäästä (julkinen pääsy, CSRF).

🔧 Testikomentoja ja Liitteitä
Käytetyt testikomennot:

bash

# Julkinen pääsy testaus

curl http://localhost:8000/resources

# CSRF-testaus

curl -X POST http://localhost:8000/resources -d "resource_name=TESTI"

# Admin-pääsyn testaus

curl -b admin_session.txt http://localhost:8000/admin
Testausympäristön tiedot:

Image: vheikkiniemi/cybersec-web-phase3:v1.1

Portit: 8003:8000 (sovellus), 5435 (PostgreSQL)

Tietokanta: PostgreSQL

Tila: Kontti pyörii normaalisti

Koetulokset yhteenvetona:

Autentikointi: 90% (toimii)

Valtuutus (Authorization): 40% (puutteellinen)

Istuntojen turvallisuus: 80% (hyvä, mutta template-bugi)

CSRF-suojaus: 30% (toimii osittain)





   









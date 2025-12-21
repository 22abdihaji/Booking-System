# Kyberturvallisuus Kurssin Loppuraportti

**Nimi:** Ali Haji  
**Päivämäärä:** 21. joulukuuta 2025

---

## PortSwigger

 Kuvakaappaukset löytyy toisesta kansiosta PortSwigger Academyn työpöydästäni, joka osoittaa suoritetut moduulit.

**Suoritetut laboratoriot (koko lista):**

### **Aihe: SQL-injektio**

- **SQL injection vulnerability in WHERE clause allowing retrieval of hidden data**
- **SQL injection vulnerability allowing login bypass**

### **Aihe: Autentikointi**

- **2FA simple bypass**
- **Password reset broken logic**

### **Aihe: Pääsynvalvonta**

- **Unprotected admin functionality**
- **User role can be modified in user profile**

---

## The Booking System -projekti

### ** Vaihe 1: Penetraatiotestaus (Phase 1)**

- **Mitä tein:** Testasin sovelluksen rekisteröintisivun turvallisuutta. Löysin SQL-injektiohaavoittuvuuksia sähköpostikentässä ja havaitsin, että lomakkeilta puuttui CSRF-suojaus.
- **Mikä toimi:** Voi nähdä selkeästi mitä ZAP-työkalu löytää.
- **Mikä ei toiminut:** Sovellus paljasti yksityiskohtaisia virheilmoituksia.
- **Mikä vei eniten aikaa:** Docker-ympäristön pystytys ja ZAP-raportin laatiminen.
- **Mitä opin:** Miten suorittaa perustason turvallisuustarkistus järjestelmällisesti ja dokumentoida löydöt ammattimaisesti.

### ** Vaihe 2: Korjausten vahvistaminen (Phase 1 - Part 2)**

- **Mitä tein:** Testasin uutta versiota, jossa kehittäjä oli yrittänyt korjata aiemmat haavoittuvuudet. Testasin, olivatko aiemmat löydöt (kuten SQL-injektio ja CSRF) korjattu.
- **Mikä toimi:** Löysin että SQL-injektio oli korjattu, mutta uusia ongelmia oli ilmaantunut.
- **Mikä ei toiminut:** CSRF-suojaus oli edelleen puutteellinen.
- **Mikä vei eniten aikaa:** Aiemman raportin löytöjen läpikäyminen ja kunkin korjauksen tilausten varmistaminen manuaalisesti.
- **Mitä opin:** Korjauksista huolimatta voi ilmaantua uusia ongelmia, ja jatkuva testaus on kriittistä.

### ** Vaihe 3: Salasanojen murtaminen (Phase 2)**

- **Mitä tein:** Katselin opetusvideon, joka paljasti järjestelmän tietokannan käyttäjät ja heidän salasanansa hash-muodossa. Käytin työkaluja (kuten hashcat) murratakseni vähintään 5 salasanaa hyödyntäen heikkoja salasanoja (kuten "password123").
- **Mikä toimi:** Sanahyökkäykset (Dictionary attacks) olivat erittäin tehokkaita, koska käyttäjät valitsivat heikkoja salasanoja.
- **Mikä ei toiminut:** Monimutkaisempia salasanoja ei voinut murtaa realistisessa ajassa ilman enemmän laskentatehoa.
- **Mikä vei eniten aikaa:** Työkalujen asennus, konfigurointi ja salasanalistojen etsiminen.
- **Mitä opin:** Kuinka tärkeää on käyttää vahvoja, yksilöllisiä salasanoja ja kuinka suuren edun hyökkääjä saa, jos pääsee käsiksi tietokannan salasanatiivisteisiin.

### ** Vaihe 4: Valtuutuksen testaus (Phase 3 - Authorization)**

- **Mitä tein:** Testasin manuaalisesti (cURL-komennoilla) eri endpointteja, kuten `/resources` ja `/admin`, sekä admin- että tavallisen käyttäjän rooleilla. Tarkistin pääsynvalvonnan ja CSRF-suojauksen.
- **Mikä toimi:** Sovelluksessa oli hyviä perusturva-asetuksia (kuten HttpOnly-evästeet, CSP).
- **Mikä ei toiminut:** Löysin **kriittisen haavoittuvuuden**: `/resources`-endpoint oli **avoin kaikille ilman kirjautumista**. Lisäksi admin-paneelia ei ollut olemassa, eikä roolipohjaista pääsynvalvontaa ollut.
- **Mikä vei eniten aikaa:** Jokaisen endpointin manuaalinen testaaminen ja istuntojen hallinta cURL:lla.
- **Mitä opin:** Kuinka tärkeää on varmistaa, että pääsynvalvonta on toteutettu loogisella tasolla (backend), ei vain käyttöliittymässä. Näin löydettiin yksi kurssin vakavimmista haavoittuvuuksista.

---

## Reflektio 

Tämä kurssi opetti minut siirtymään teoreettisesta tietoturvatiedosta käytännön testaamiseen. Suurin oppiminen oli se, että turvallisuusvauriot löytyvät usein perusasioista, kuten puuttuvasta pääsynvalvonnasta tai heikoista salasanoista. Nyt ymmärrän, miten tärkeää on ajatella hyökkääjän näkökulmasta: testata kaikkea, mitä järjestelmä sallii, ei vain sitä, mitä sen pitäisi sallia. Käytännön työkalut kuten OWASP ZAP ja manuaalinen testaaminen cURL:lla antavat konkreettiset taidot, joita tarvitsen tulevissa tehtävissä.

---

## Logbook

**Linkki GitHub-repositoriooni:**  
[https://github.com/22abdihaji/Booking-System/blob/main/logbook.md) 

**Käytetyt tunnit yhteensä:** **49 tuntia**

**Käytetyt tunnit aiheittain:**  **3 tuntia - 7 tuntia**

| Aihe                            |    Tunnit     |
| :------------------------------ | :-----------: |
| **PortSwigger-laboratoriot**    |   16 tuntia   |
| **Booking System -projekti**    |   22 tuntia   |
| &nbsp;&nbsp;• Vaihe 1           |   5 tuntia    |
| &nbsp;&nbsp;• Vaihe 1, Part 2   |   4 tuntia    |
| &nbsp;&nbsp;• Vaihe 2           |   5 tuntia    |
| &nbsp;&nbsp;• Vaihe 3           |   4 tuntia    |
| **Dokumentointi & raportointi** |   4 tuntia    |
| **YHTEENSÄ**                    | **56 tuntia** |

---

## Feedback 

Kurssi tarjosi erinomaisen käytännönläheisen johdatuksen penetraatiotestaukseen. Booking System -projekti oli erityisen hyödyllinen, sillä se simuloi aidosti tyypillistä turvallisuustarkistuksen työnkulkua aina haavoittuvuuksien etsinnästä raportointiin. OWASP ZAP:n käyttö ja manuaalisen testauksen opettelu olivat arvokkaita taitoja. Ainoa parannusehdotus olisi enemmän aikaa ja mahdollisesti yksi ylimääräinen vaihe, joka keskittyisi XSS- tai SS-haavoittuvuuksiin.

---








# ryhti_building_address
SyKe Ryhti dataa ja skriptejä validointiin yms

# errata tiedostot

Ajettu erilaisia tarkistuksia ryhti datalle.

- Löydetyt vanhentuneet kuntakoodit löytyvät tiedostosta errata.municipality_number

- errata tiedostot ovat muodostuneet useasta aineistosta.
  - SyKe ryhti building ja address aineisto
  - MML Väestötietojärjestelmän rakennustietokannasta, https://www.maanmittauslaitos.fi/palvelutiedotteet/rakennusten-kyselypalvelu-ogc-api-features-poistuu-kaytosta-kevaalla-2025
  - MML Maastotietokannan rakennus ja rakennelma -tietokannasta
  - Väyläviraston viitekehysmuuntimella on haettu rakennuksille osoite.
  - MML Inspire kuntadata
 
- korjaustiedot on ajettu seuraavasti:
  1) Koordinaatin perusteella mätsätty ryhti koordinaatti MML maastotietokannan rakennuksiin (2566007 kpl)
  2) Korjattu mahdollinen virhe koordinaatissa ja pysyvässärakennustunnuksessa
  3) Ilman koordinaattia haettu pysyvällä rakennustunnuksella ja korjattu koordinaatti sekä mahdollisesti kiinteistötunnus (657644 kpl)
  4) Etsitty osoitteella ja heuristinen arvaus parhaasta rakennuksesta ko. kiinteistöllä (18469 kpl)
  5) Kiinteistön perusteella löydetty ja heuristinen arvaus parhaasta rakennuksesta ko. kiinteistöllä (438745 kpl)
  6) Koordinaatin perusteella lohkottu kiinteistö, entinen kiinteistö löytyy läheltä. Heuristinen arvaus lähimmästä rakennuksesta (2257 kpl)

- 36987 kpl rakennuksia, joille ei löytynyt sopivaa kiinteistöä tai rakennusta.

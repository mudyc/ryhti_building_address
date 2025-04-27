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
  - MML Inspire kuntadata ja kiinteistödata
 
- korjaustiedot on ajettu seuraavasti. Tässä ongelmana on, että mihinkään tietoon ei voi luottaa. Osa koordinaateista ihan höpöä. Osa kiinteistötiedoista höpöä. Lähdetty purkamaan tilannetta niin, että pysyvä rakennustunnus on oletettu oikeaksi. Tämän jälkeen rakennuksen osoite (viitekehysmuuntimesta). Sitten kiinteistötunnus (jos kiinteistöltä löytyy rakennuksia). Tämän jälkeen palattu koordinaatin lähietsintöihin.
  1. Koordinaatin perusteella mätsätty ryhti koordinaatti MML maastotietokannan rakennuksiin (2´566´007 kpl)
  2. Korjattu mahdollinen virhe koordinaatissa ja pysyvässärakennustunnuksessa (laskettu seuraavaan kohtaan)
  3. Ilman koordinaattia haettu pysyvällä rakennustunnuksella ja korjattu koordinaatti sekä mahdollisesti kiinteistötunnus (657´644 kpl)
  4. Etsitty osoitteella ja heuristinen arvaus parhaasta rakennuksesta ko. kiinteistöllä (18´469 kpl)
  5. Kiinteistön perusteella löydetty ja heuristinen arvaus parhaasta rakennuksesta ko. kiinteistöllä (438´745 kpl)
  6. Koordinaatin perusteella lohkottu kiinteistö, entinen kiinteistö löytyy läheltä. Heuristinen arvaus lähimmästä rakennuksesta (2´257 kpl)
  7. Koordinaatin perusteella kiinteistö löytyy läheltä (6`401 kpl)
  8. Suodatettu tuulivoimalat pois (1´141 kpl)
  9. Kerätty loput käsintarkistettaviksi:
      a. Osoitteettomat (7´530 kpl)
      b. Rakennus purettu? (3´435 kpl)
      c. Muut (19´064 kpl)


# Heuristinen mätsäys kiinteistön sisällä

Ilmeisesti osa rakennusten koordinaateista on paljonkin pielessä. Osa koordinaateista osoittaa väärään
rakennukseen, osa rakennuksen viereen, osa kiinteistön keskelle, osa viereiseen kiinteistöön. Oheisella
koodilla pyrittiin yhdistämään MML maastotietokannan rakennustieto (esim. käyttötarkoitus 
https://xml.nls.fi/XML/Schema/Maastotietojarjestelma/MTK/202203/Koodistot/RakennuksenKayttotarkoitus.xsd) 
ja rakennuksen pohja-ala SyKe Ryhti rakennustietoon.

function probabilityMatchScore(feature, candidate) {
    function mapMainPurpose(uri) {
        if (!uri) return null;
      
        const code = uri.split('/').pop();
        switch (code) {
          case '01': return 3; // Vapaa-ajan asuinrakennus => Lomarakennus
          case '02': return 4; // Toimisto/tuotanto => Teollinen rakennus
          case '03': return 6; // Talousrakennus => Muu rakennus
          case '04': return 6; // Saunarakennus => Muu rakennus
          case '05': return 1; // Pientalo => Asuinrakennus
          case '06': return 1; // Kerrostalo => Asuinrakennus
          case '07': return 2; // Julkinen rakennus
          default: return 7;   // Luokittelematon
        }
    }

    const distance = pointToObjectDistance(feature, candidate);
    const distanceProb = Math.exp(-distance / 100); // mitä lähempänä, sitä parempi
  
    const featurePurpose = mapMainPurpose(feature.properties.main_purpose);
    const purposeProb = candidate.kayttotarkoitus === featurePurpose ? 1: 0;
  
    const kerrosProb = 
        ((candidate.properties.kerrosluku == 1 && [1,2].includes(feature.properties.number_of_storeys))
        || (candidate.properties.kerrosluku == 2 && feature.properties.number_of_storeys >= 3)) ? 1: 0

    const area = feature.properties.gross_floor_area;
    const candidateArea = calculatePolygonArea(candidate);

    const areaRatio = Math.min(area, candidateArea) / Math.max(area, candidateArea);
    const areaProb = areaRatio; // suora suhde
    
    // Painotettu yhdistelmä
    const score =
      0.3 * distanceProb +
      0.2 * kerrosProb +
      0.2 * areaProb +
      0.2 * purposeProb;
  
    return { candidate, score };
}

function findMostProbableMatch(feature, candidates) {
    if (candidates.length == 1) return { candidate: candidates[0], score: 1 }

    const scored = candidates.map(c => probabilityMatchScore(feature, c));
    scored.sort((a, b) => b.score - a.score); // suurin todennäköisyys ensin
    return scored[0]; // paras ehdokas
}




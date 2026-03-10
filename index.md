# Vesimittarin kulutusseuranta Home Assistantissa

Monissa kiinteistöissä vesimittarit lähettävät kulutus- ja tilatietoja langattomasti etäluentaa varten. Halusin hyödyntää tätä, jotta voin seurata omaa vedenkulutusta ja mittarin tilatietoja.
Tavoitteena oli myös toteuttaa mikrokontrollerilla oikeasti hyödyllinen “end-to-end” projekti, jossa data kulkee kenttälaitteelta seurantaan asti.

## Käytetyt teknologiat projektissa

- **wM-Bus (Wireless M-Bus):** vesimittarin käyttämä langaton tiedonsiirtoprotokolla, jonka telegrammeja projektissa käsiteltiin.
- **CC1101 (868 MHz):** radiomoduuli, jolla vesimittarin langaton wM-Bus-signaali vastaanotettiin.
- **ESP32:** mikrokontrolleri, joka ohjasi CC1101-radiomoduulia, käsitteli vastaanotetut wM-Bus-telegrammit ja välitti ne eteenpäin verkon yli.
- **Home Assistant (HA):** avoimen lähdekoodin kotiautomaatioalusta, johon vesimittarin viestit tuotiin käsiteltäviksi. Home Assistantissa raakadata muunnettiin käyttöliittymässä näkyviksi mittausarvoiksi ja tilatiedoiksi, joiden pohjalta rakennettiin kulutus- ja kustannusseuranta sekä mittarin tila- ja hälytysseuranta dashboardille.
- **MQTT:** kevyt viestinvälitysprotokolla, jolla ESP32 lähetti vastaanotetut telegrammit Home Assistantiin Mosquitto-brokerin kautta.

## Tulokset

### Home Assistant -dashboard: kulutus, kustannus ja mittarin tilat
Alla on kuvakaappaus Home Assistantin dashboard-näkymästä, johon koottiin vesimittarin tärkeimmät kulutus-, kustannus- ja tilatiedot:
- **kuukausi-, päivä-** sekä **kuluvan päivän huippukulutus**
- **kustannusseuranta** (veden hinta määritelty HA:ssa)
- **mittarin tilatiedot, lämpötilat ja hälytykset** (esim. jatkuva vuoto / takaisinvirtaus)
![Vedenkulutus dashboard](HA-water-dashboard.png)

### Home Assistant Energy -näkymä: tuntiprofiili ja seuranta
Home Assistantin Energy-näkymässä kulutus näkyy selkeästi **tuntitasolla**, jolloin:
- suihkut ja muut kulutuspiikit erottuvat helposti
- kulutuksen rytmiä voi verrata eri päivien välillä
- näkymä toimii myös “päästä päähän” -varmistuksena: kulutus kertyy ajan myötä ja mittausarvot näkyvät tasaisena kertymänä
![Tuntikulutus ja kustannus](water-consumption-graph.png)

## Toteutus lyhyesti

### 1) Laitteisto ja vastaanotto (wM-Bus 868 MHz)
Vesimittarin wM-Bus-telegrammit vastaanotetaan **radiosignaalina** 868 MHz taajuudella **CC1101**-radiomodulilla, joka on kytketty **ESP32**-mikrokontrolleriin (ESP32-WROOM-32E) SPI-väylällä. Koodasin ESP32:n vastaanotto- ja välitysohjelman itse **Arduino-ympäristössä**.
<p align="center">
  <img src="ESP32.png" alt="ESP32"
       style="width:60%; max-width:420px; height:auto;">
  <br>
  <em>Käyttämäni ESP-32 mikrokontrolleri</em>
</p>

### 2) Siirto Home Assistantiin (MQTT)
ESP32 julkaisee vastaanotetun **wM-Bus-telegrammin payloadin** MQTT:llä Home Assistant -ympäristöön. MQTT perustuu julkaisu–tilaus -malliin (pub/sub), ja brokerina toimii Home Assistantin **Mosquitto broker** -lisäosa.

### 3) Purku (wmbusmeters)
Home Assistantissa **wmbusmeters-lisäosa** kuuntelee MQTT-topiccia, purkaa wM-Bus-telegrammin ja hoitaa myös **salauksen purun**. Tämän jälkeen mittausarvot muodostuvat sensoreiksi (esim. kokonaiskulutus, virtaus, lämpötilat sekä mittarin tila-/hälytysbitit).

### 4) Kulutusseuranta ja valvonta
Home Assistantissa rakensin sensoreiden pohjalta:
- **tunti-/päivä-/kuukausikulutuksen**
- **kustannusseurannan**
- **mittarin tila- ja hälytystietojen** seurannan dashboardissa

## Testaus ja vianrajaus (päästä päähän)

- **SPI-perustesti (CC1101):** ennen radiovastaanoton testausta varmistin ESP32:n ja CC1101:n välisen SPI-yhteyden ajamalla toistuvaa SPI-testiä, joka lukee CC1101:n PARTNUM- ja VERSION-rekisterit. Tällä varmistin, että johdotus ja SPI-kommunikaatio toimivat vakaasti.
- **Radiovastaanotto:** varmistin ESP32:n debug-/lokitulosteista, että telegrammeja tulee odotetulla tahdilla ja että ne vastaavat omaa mittaria.
- **Purku paikallisesti:** testasin salauksen purun ja telegrammien lukemisen ajamalla **wmbusmeters**-ohjelmaa omassa ympäristössä ennen Home Assistant -integraatiota.
- **MQTT-siirto:** varmistin, että viestit näkyvät brokerilla ja että Home Assistant vastaanottaa ne.
- **HA-sensorit ja lopputulos:** varmistin wmbusmeters-lisäosan lokien avulla, että telegrammien purku onnistuu. Tämän jälkeen tarkistin, että wmbusmeters luo oikeat Home Assistant -sensorit. Lopuksi seurasin dashboard- ja Energy-näkymiä ja varmistin, että sensorien arvot vastaavat odotettua kulutusta.
- **Jatkokehitys (diagnostiikan parantaminen):** MQTT:n kautta voisi lähettää erillisen heartbeat/keep-alive-viestin, jolloin ESP32:n online/offline-tila näkyy Home Assistantissa. Tämän perusteella voi tehdä Home Assistantissa hälytyksen (esim. sähköposti/tekstiviesti), jos laite putoaa pois.
<link rel="stylesheet" href="assets/css/custom.css">

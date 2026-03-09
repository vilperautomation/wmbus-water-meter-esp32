---
layout: page
title: "Vesimittarin wM-Bus-datan vastaanotto ja kulutusseuranta Home Assistantissa"
---

**Tavoite:** automaattinen vesikulutuksen seuranta (tunti/päivä/kk), kustannuslaskenta sekä mittarin tilojen/hälytysten valvonta.  
**Teknologiat:** ESP32, CC1101 (868 MHz), wM-Bus, MQTT, Home Assistant + wmbusmeters

Monissa kiinteistöissä vesimittarit lähettävät kulutus- ja tilatietoja langattomasti etäluentaa varten. Halusin hyödyntää tätä, jotta voin seurata omaa vedenkulutusta ja mittarin tilatietoja Home Assistantissa.
Rakensin mittausketjun: vesimittarin wM-Bus-telegrammit vastaanotetaan 868 MHz CC1101-radiolla, ESP32 välittää ne MQTT:llä Home Assistantiin ja wmbusmeters-lisäosa purkaa ne sensoreiksi. Home Assistantissa rakensin kulutus- ja kustannusseurannan sekä mittarin tila-/hälytysseurannan dashboardille.

## Tulokset

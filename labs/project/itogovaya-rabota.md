---
description: >-
  Построение и объединение двух ЦОД на основе технологии VxLAN EVPN на
  оборудовании Cisco Nexus
---

# Итоговая работа

#### **Цели**

* Реализация сетевой фабрики с использованием Overlay технологии VxLAN EVPN
* Объединение двух ЦОД-ов (POD) на основе Multi-Pod топологии
* Протестировать связанность между клиентами внутри VNI, между VNI внутри одного VRF, а также между VNI в разных VRF.

#### **Задачи**

* Реализация сетевой фабрики с использованием Overlay технологии VxLAN EVPN
* Реализация сетевой фабрики с использованием Overlay технологии VxLAN EVPN
* Реализация сетевой фабрики с использованием Overlay технологии VxLAN EVPN



#### Используемая схема сети на основе коммутаторов Nexus 9000/9300 Series:



#### Конфигурация устройств

<details>

<summary>LEAF1</summary>



</details>

<details>

<summary>LEAF1</summary>



</details>

<details>

<summary>LEAF1</summary>



</details>

<details>

<summary>LEAF1</summary>



</details>

<details>

<summary>LEAF1</summary>



</details>

<details>

<summary>LEAF1</summary>



</details>

<details>

<summary>LEAF1</summary>



</details>

#### Проверка

ipv4 и l2vpn evpn маршруты

LEAF1



Проверка связности между endhost-1, находящимся в VRF COD, и endhost-2, endhost-3, находящимися в VRF TENANT\_1, а также Loopback интерфейсом на EDGE роутере



#### Дампы Wireshark







#### Выводы

Таким образом, нам удалось настроить ликинг между VRF COD и VRF TENANT\_1 с использованием ipv4 BGP сессий в соответствующих vrf между LEAF3 и EDGE роутером, находящимся в другом логическом сегменте.

Маршруты появились на LEAF3 как в ipv4, так и в l2vpn evpn таблицах.&#x20;

Связность между узлами сети имеется.

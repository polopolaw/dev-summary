Registry -  это stateless, high scalable серверное приложение, которое хранит и  распространяет ваши Docker образы.

Это может быть dockerhub, aws, gitlab...
Его удобней использовать чем собирать все на месте на сервере.

Registry стоит использовать, когда вы хотите:
- Тщательно контролировать, где и как хранятся ваши образы.
- Организовать свой собственный images distribution pipeline
- Интегрировать хранение  и дистрибьюцию глубоко в ваш in-house development  workflow
- Вообще-то всегда 🙃


Схема работы выглядит так:
![[Pasted image 20240925115534.png]]Туториал по созданию local docker registry
https://k21academy.com/docker-kubernetes/how-to-set-up-your-own-local-docker-registry-a-step-by-step-guide/

Сначала на одной из машин мы собираем образ из Dockerfile - образ пушите в Registry - там где необходимо образ пулится и создается контейнер. 
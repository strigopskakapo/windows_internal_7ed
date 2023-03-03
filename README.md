# Windows Internals 7ed 

## Chapitre 1 - Concepts et outils
##### Windows API
___
L'API Windows a commencé avec des fonctions de style C car elles étaient faciles d'accès depuis d'autres langages et pouvaient exposer les services du système d'exploitation. Cependant, le grand nombre de fonctions et le manque de dénomination et de regroupement logique cohérents ont rendu difficile leur utilisation par les développeurs. Pour résoudre ce problème, de nouvelles API ont commencé à utiliser le Component Object Model (COM).

Le COM a été créé à l'origine pour aider les applications Microsoft Office à communiquer et à échanger des données entre des documents. Il fonctionne en permettant aux clients de communiquer avec des objets (également appelés objets serveurs COM) via des interfaces, qui sont des contrats bien définis avec un ensemble de méthodes connexes. Cela permet une compatibilité binaire, ce qui signifie que les méthodes peuvent être appelées à partir de nombreux langages et compilateurs différents.

Un autre principe du COM est que la mise en œuvre des composants est chargée dynamiquement plutôt que d'être liée statiquement au client. Cela signifie que le code des classes COM n'est pas inclus dans l'application cliente elle-même, mais est plutôt chargé lorsque nécessaire. Le COM possède également des fonctionnalités importantes liées à la sécurité, à la communication inter-processus, aux threads, et plus encore.

Dans l'ensemble, le COM est un moyen pour les développeurs de créer des composants logiciels réutilisables pouvant être utilisés dans différentes applications et langages de programmation.

##### Windows Runtime
___
Windows 8 a introduit une nouvelle API et runtime appelé le Windows Runtime (WinRT) destiné aux développeurs d'applications pour Windows, pouvant cibler différents facteurs de forme de périphériques. WinRT est construit sur COM et ajoute diverses extensions, notamment des métadonnées de type disponibles dans les fichiers WINMD. Les API WinRT sont plus cohérentes et ont des dénominations et des modèles de programmation cohérents par rapport aux fonctions classiques de l'API Windows. Les Windows Apps (Apps Store) sont soumises à de nouvelles règles, contrairement aux applications classiques de Windows, et peuvent utiliser un sous-ensemble des API Win32 et COM. Les applications de bureau peuvent également utiliser un sous-ensemble des API WinRT. Les développeurs peuvent consommer facilement les API WinRT en utilisant des projections de langage développées pour C++, C#, .NET et JavaScript. C++ a une extension non standard appelée C++/CX, .NET a une couche d'interopérabilité COM, et JavaScript a une extension appelée WinJS.

##### .NET
___

Le **Microsoft .NET Framework** se compose d'une bibliothèque de classes appelée **Framework Class Library (FCL)** et d'un **Common Language Runtime (CLR)** qui fournit un environnement d'exécution de code géré avec des fonctionnalités telles que la compilation juste-à-temps, la vérification de type, la collecte des déchets et la sécurité d'accès au code.

Framework Class Library : Le FCL fournit un ensemble de fonctionnalités communes pouvant être utilisées par n'importe quelle application .NET, telles que des classes pour travailler avec des collections, gérer les entrées et sorties de fichiers et gérer les connexions réseau. En utilisant le FCL, les développeurs peuvent éviter d'avoir à écrire leur propre code de bas niveau pour gérer ces tâches, ce qui peut leur faire gagner du temps et des efforts. Le FCL est organisé en espaces de noms, qui regroupent des classes et des types connexes. Par exemple, l'espace de noms System.IO contient des classes pour travailler avec des fichiers et des répertoires, tandis que l'espace de noms System.Net contient des classes pour travailler avec des connexions réseau. L'un des avantages du FCL est qu'il est conçu pour être indépendant de la plate-forme, ce qui signifie que les applications développées à l'aide du Framework .NET peuvent s'exécuter sur n'importe quelle plate-forme prenant en charge le framework, y compris Windows, macOS et Linux. Cela facilite la création d'applications multiplateformes qui peuvent s'exécuter sur plusieurs systèmes d'exploitation sans avoir à écrire de code spécifique à la plate-forme.

Common Language Runtime : L'une des fonctions principales du CLR est de fournir un environnement d'exécution de code géré pour les applications .NET. Lorsqu'une application est exécutée sur le CLR, celui-ci est responsable de la gestion de l'allocation et de la désallocation de la mémoire, de la collecte des déchets et de la sécurité. Cela permet aux développeurs de se concentrer sur l'écriture de code sans avoir à se soucier des détails de bas niveau tels que la gestion de la mémoire. Le CLR fournit également un compilateur juste-à-temps (JIT), qui compile le code .NET en code machine au moment de l'exécution. Cela permet au CLR d'optimiser le code pour la plate-forme matérielle spécifique sur laquelle l'application s'exécute, ce qui peut améliorer les performances.

> :memo: Le compilateur .NET convertit le code source en une forme intermédiaire (bytecode) appelée Common Intermediate Language (CIL), qui est ensuite traité par le CLR pour produire du code natif [1].

> :memo: Le CLR est implémenté en tant que serveur COM classique dont le code réside dans une DLL Windows standard en mode utilisateur. Rien de la plateforme .NET ne s'exécute en mode noyau [2].

> :memo: Tous les composants du framework Dotnet sont implémentés en tant que DLL Windows standard en mode utilisateur (rien ne s'exécute en mode noyau) [2].

> :memo: Le code géré (code exécuté par le CLR) et le code non géré (par exemple, les API de code natif, telles que Win32) peuvent coexister dans une même application [1].
<p align="center">
<img src="https://i.imgur.com/3v7aTwK.png" )
</p>

##### Services, functions and routines
___
Plusieurs termes dans la documentation utilisateur et de programmation de Windows ont des significations différentes dans différents contextes. Par exemple, le mot "service" peut faire référence à une routine appelable dans le système d'exploitation, un pilote de périphérique ou un processus de serveur. La liste suivante décrit ce que certains termes signifient dans ce livre :

- Windows API functions : Ce sont des sous-routines appelables documentées dans l'API Windows. Des exemples incluent CreateProcess, CreateFile et GetMessage.
- Native system services (ou system calls) : Ce sont les services sous-jacents non documentés dans le système d'exploitation qui sont appelables à partir du mode utilisateur. Par exemple, NtCreateUserProcess est le service système interne que la fonction Windows CreateProcess appelle pour créer un nouveau processus.
- Kernel support functions (ou routines) : Ce sont les sous-routines à l'intérieur du système d'exploitation Windows qui ne peuvent être appelées qu'à partir du mode noyau (défini plus loin dans ce chapitre). Par exemple, ExAllocatePoolWithTag est la routine que les pilotes de périphériques appellent pour allouer de la mémoire à partir des tas système Windows/Windows system heaps (appelés pools).
- Windows services : Il s'agit de processus lancés par le gestionnaire de contrôle de service Windows. Par exemple, le service Planificateur de tâches s'exécute dans un processus en mode utilisateur qui prend en charge la commande schtasks (qui est similaire aux commandes UNIX at et cron). (Notez que bien que le registre définisse les pilotes de périphériques Windows comme "services", ils ne sont pas désignés comme tels dans ce livre.)
- Dynamic link libraries (DLL) : Ce sont des sous-routines appelables liées ensemble sous la forme d'un fichier binaire qui peut être chargé dynamiquement par des applications qui utilisent ces sous-routines. Des exemples incluent Msvcrt.dll (la bibliothèque d'exécution C) et Kernel32.dll (l'une des bibliothèques de sous-système de l'API Windows). Les composants et les applications en mode utilisateur de Windows utilisent largement les DLL. L'avantage des DLL par rapport aux bibliothèques statiques est que les applications peuvent partager les DLL, et Windows veille à ce qu'il n'y ait qu'une seule copie en mémoire du code d'une DLL parmi les applications qui y font référence. Notez que les assemblies de la bibliothèque .NET sont compilées en tant que DLL mais sans aucune sous-routine exportée non gérée. Au lieu de cela, le CLR analyse les métadonnées compilées pour accéder aux types et membres correspondants. [1]

Dans le contexte de la documentation de Windows, le terme "service" peut faire référence à plusieurs choses différentes, y compris des fonctions d'API documentées, des services système natifs non documentés, des fonctions de support du noyau, des services Windows lancés par le gestionnaire de contrôle de service Windows, et des bibliothèques de liens dynamiques (DLL).

##### Processes 
____

Au plus haut niveau d'abstraction, un processus Windows comprend ce qui suit :

- Un espace d'adressage virtuel privé (VAS - Virtual Address Space), qui est un ensemble d'adresses de mémoire virtuelle que le processus peut utiliser.
- Un programme exécutable, qui définit le code et les données initiales et est mappé dans le VAS du processus.
- Une liste de descripteurs ouverts vers diverses ressources système - telles que des sémaphores, des ports de communication et des fichiers - accessibles à tous les threads du processus.
- Un contexte de sécurité appelé jeton d'accès qui identifie l'utilisateur, les groupes de sécurité, les privilèges, l'état de virtualisation de contrôle de compte d'utilisateur (UAC), la session et l'état de compte utilisateur limité associé au processus.
- Un identificateur unique appelé ID de processus (interne à un identificateur appelé ID de client).
Au moins un thread d'exécution (bien qu'un processus "vide" soit possible, il n'est pas utile).

Chaque processus pointe également vers son processus parent ou créateur. Si le parent n'existe plus, cette information n'est pas mise à jour.
Windows ne maintient que l'ID du processus créateur, pas de lien vers le créateur du créateur, et ainsi de suite.

À noter qu'un processus dans l'état "ne répond pas" signifie que le thread pourrait être en cours d'exécution ou en attente d'une entrée/sortie ou d'un objet de synchronisation Windows.

##### Threads
___
Un thread comprend les composants essentiels suivants :
- Un ensemble de registres de processeur représentant l'état du processeur
deux piles - l'une que le thread utilise lors de l'exécution en mode kernel et l'autre pour l'exécution en mode utilisateur
- Une zone de stockage privée appelée thread-local storage (TLS) à utiliser par les sous-systèmes, les bibliothèques d'exécution et les DLL
- Un identificateur unique appelé ID de thread (partie d'une structure interne appelée ID de client - les IDs de processus et de threads sont générés à partir du même espace de noms, de sorte qu'ils ne se chevauchent jamais)
- Les threads ont parfois leur propre contexte de sécurité ou jeton, souvent utilisé par les applications serveur multithreadées qui se font passer pour le contexte de sécurité des clients qu'elles servent.

Les registres volatils, les piles et la zone de stockage privée sont appelés le contexte du thread (Utilisez GetThreadContext() ou Wow64GetThreadContext() pour obtenir le contexte.)

La commutation d'exécution entre les threads implique l'ordonnanceur de noyau et peut être une opération coûteuse. Pour réduire ce coût, Windows implémente deux mécanismes : **fibers** et **user-mode scheduling (UMS)**.

> :memo: Les threads d'une application 32 bits s'exécutant sur une version 64 bits de Windows contiendront à la fois des contextes 32 bits et 64 bits, que Wow64 (Windows on Windows) utilisera pour basculer l'application de l'exécution en mode 32 bits à 64 bits lorsque nécessaire. Ces threads auront deux piles utilisateur et deux blocs CONTEXT, et les fonctions habituelles de l'API Windows retourneront le contexte 64 bits. Toutefois, la fonction Wow64GetThreadContext retournera le contexte 32 bits. Voir le chapitre 8 de la partie 2 pour plus d'informations sur Wow64.

## Fibers
____

Les "user-mode threads" (threads en mode utilisateur) permettent à une application de planifier ses propres "threads" d'exécution plutôt que de s'appuyer sur le mécanisme de planification basé sur les priorités intégré à Windows. Ils sont plus légers et invisibles pour le noyau car ils sont implémentés en mode utilisateur dans Kernel32.dll. On peut les comparer aux "goroutines" en Go.

## User-mode scheduling threads
___

User-mode scheduling (UMS) threads est uniquement disponible sur les versions 64 bits de Windows. Fournissent les mêmes avantages de base que les fibres, sans beaucoup de leurs inconvénients.

Ont leur propre état de thread du noyau et sont donc visibles du noyau, ce qui permet à plusieurs threads UMS d'émettre des appels système bloquants, de partager et de lutter pour les ressources et d'avoir un état par thread.

Cependant, tant que deux ou plusieurs threads UMS ont seulement besoin d'effectuer un travail en mode utilisateur, ils peuvent périodiquement basculer les contextes d'exécution (en cédant d'un thread à un autre) sans impliquer le planificateur : la commutation de contexte est effectuée en mode utilisateur.
Du point de vue du noyau, le même thread du noyau est toujours en cours d'exécution et rien n'a changé. Lorsqu'un thread UMS effectue une opération nécessitant l'entrée dans le noyau (comme un appel système), il bascule vers son thread du noyau dédié (appelé commutation de contexte dirigée).

Tous les threads dans un processus ont un accès complet en lecture-écriture à l'espace d'adressage virtuel du processus.
Les threads ne peuvent pas accidentellement référencer l'espace d'adressage d'un autre processus, sauf si l'autre processus met à disposition une partie de son espace d'adressage privé en tant que section de mémoire partagée (appelée objet de mappage de fichier dans l'API Windows) ou si un processus a le droit d'ouvrir un autre processus pour utiliser des fonctions de mémoire interprocessus telles que ReadProcessMemory() et WriteProcessMemory().

En plus d'un espace d'adressage privé et d'un ou plusieurs threads, chaque processus a un contexte de sécurité
et une liste de poignées ouvertes vers des objets du noyau tels que des fichiers, des mutex, des événements ou des sémaphores ...

<p align="center"><img src="https://i.imgur.com/MSIUr1C.png" width="400px" height="auto"></p>

Les virtual address descriptors (VADs) sont des structures de données que le gestionnaire de mémoire utilise pour suivre les adresses virtuelles que le processus utilise.

Par défaut, les threads n'ont pas leur propre jeton d'accès, mais ils peuvent en obtenir un, ce qui permet à chaque thread d'usurper le contexte de sécurité d'un autre processus, y compris des processus sur un système Windows distant, sans affecter les autres threads du processus.

## Jobs
___

Windows fournit une extension au modèle de processus appelé un "job".
La principale fonction d'un objet de travail est de permettre la gestion et la manipulation de groupes de processus en tant qu'unité. En quelque sorte, l'objet de travail compense l'absence d'arbre de processus structuré dans Windows, mais dans bien des cas, il est plus puissant qu'un arbre de processus de style UNIX.

## Virtual Memory
___

Windows utilise un système de mémoire virtuelle basé sur un espace d'adressage plat (linéaire) qui fournit à chaque processus l'illusion d'avoir son propre grand espace d'adressage privé.
À l'exécution, le gestionnaire de mémoire, avec l'aide du matériel, traduit ou mappe les adresses virtuelles en adresses physiques, où les données sont réellement stockées.
En contrôlant la protection et le mapping, le système d'exploitation peut s'assurer que les processus individuels ne se heurtent pas les uns aux autres ou n'écrasent pas les données du système d'exploitation.

<p align="center"><img src="https://i.imgur.com/K8c0GAm.png" width="400px" height="auto"></p>

Étant donné que la plupart des systèmes ont beaucoup moins de mémoire physique que la mémoire virtuelle totale utilisée par les processus en cours d'exécution, le gestionnaire de mémoire transfère ou pagine certaines parties de la mémoire vers le disque.

La pagination des données sur le disque libère de la mémoire physique pour qu'elle puisse être utilisée par d'autres processus ou par le système d'exploitation lui-même. Lorsqu'un thread accède à une adresse virtuelle qui a été paginée sur le disque, le gestionnaire de mémoire virtuelle charge les informations de nouveau en mémoire depuis le disque.

Les applications n'ont pas besoin d'être modifiées de quelque manière que ce soit pour profiter de la pagination, car le support matériel permet au gestionnaire de mémoire de paginer sans la connaissance ou l'aide des processus ou des threads.

La taille de l'espace d'adressage virtuel varie pour chaque plateforme matérielle.

Sur un x86 32 bits, un processus peut adresser un espace de mémoire de 4 Go.

Par défaut, Windows alloue la moitié de cet espace d'adressage (la moitié inférieure de l'espace d'adressage virtuel de 4 Go, de __0x00000000 à 0x7FFFFFFF__) aux processus pour leur stockage privé unique.
et utilise l'autre moitié (la moitié supérieure, les adresses de __0x80000000 à 0xFFFFFFFF__) pour son propre usage de mémoire OS protégée.

Windows prend en charge des options de démarrage (le qualificateur increaseuserva dans la base de données de configuration de démarrage) qui donnent aux processus exécutant des programmes spécialement marqués (le drapeau de grand espace d'adressage doit être défini dans l'en-tête de l'image exécutable) la capacité d'utiliser jusqu'à 3 Go d'espace d'adressage privé (en laissant 1 Go pour le noyau).

Bien que 3 Go soient meilleurs que 2 Go, ce n'est toujours pas suffisant pour mapper des bases de données très volumineuses (multigigaoctets).
Pour répondre à ce besoin sur les systèmes 32 bits, Windows fournit un mécanisme appelé Address Windowing Extension (AWE), qui permet à une application 32 bits d'allouer jusqu'à 64 Go de mémoire physique, puis de mapper des vues, ou fenêtres, dans son espace d'adressage virtuel de 2 Go.

Bien que l'utilisation d'AWE impose au programmeur la charge de gérer les mappings de la mémoire virtuelle à la mémoire physique, elle répond au besoin d'accéder directement à plus de mémoire physique que ce qui peut être mappé à un moment donné dans l'espace d'adressage des processus 32 bits.

Windows 64 bits fournit un espace d'adressage beaucoup plus grand pour les processus :
- 7152 Go sur les systèmes IA-64
et 8192 Go sur les systèmes x

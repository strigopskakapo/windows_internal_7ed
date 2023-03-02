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

<div style="text-align: center;">

![Relations between .NET and the Windows OS](https://i.imgur.com/3v7aTwK.png)

</div>

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

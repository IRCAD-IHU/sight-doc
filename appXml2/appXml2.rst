AppXml2
****************************************

Ce document résume les modifications proposées dans le cadre des travaux qui ont été regroupés sous la bannière "AppXml2". 
En réalité il s'agit davantage d'une refonte de l'approche objets/services qui a ensuite été appliquée à un nouveau type d'application Xml.

1. Description de l'existant
===========================================

1.1. Un service travaille sur une donnée
-------------------------------------------

Notre approche objets/services repose sur le postulat qu'un service travaille sur une unique donnée. Si cela fonctionne parfaitement sur des cas simples, comme les readers ou writers, il nous arrive très rapidement de devoir dépasser cette contrainte dès qu'une application entière est construite. Bien souvent il n'est pas possible, ou pas souhaitable de découper un algorithme travaillant sur N données en N services.

Ce constat historique a mené à l'introduction d'une nouvelle donnée, le **composite**, contenant une map d'objets. Les données nécessaires au service sont introduites dans cette collection. Le service décrit ensuite quelles clés il utilise dans cette donnée. Nous nous retrouvons rapidement avec un composite partagé par plusieurs services avec de nombreuses clés. Pour isoler certains services (même si ce n'est pas l'unique raison), des composites peuvent même être ajoutés dans les composites. La lecture des XML devient pénible avec de nombreuses imbrications et conduit également à des jonglages de données entre les composite pour faire fonctionner le tout correctement.

Nous l'aurons compris toute application hors des cas triviaux comme les tutoriels utilise donc inévitablement un composite comme donnée principale et de nombreux services travaillent sur des composites, donc des données multiples.

1.2. Récupération des données et connexions
----------------------------------------------

L'implication en terme de développement d'un service est loin d'être anodine. Pour chaque service, il faut déclarer quelles clés utiliser dans la configuration du service. Il faut donc parser une string dans l'implémentation de *IService::configuring()*, la stocker puis récupérer la donnée associée dans le composite dans *IService::starting()*. 

Ensuite pour être notifié des changements, il faut créer les connexions à la main, indépendamment de l'attribut autoConnect du service, car la fonction virtuelle *IService::getObjSrvConnections()* ne travaille que sur le composite lui-même. Entre un service qui travaille directement sur une image ou un service qui travaille sur une image dans un composite, il y aura donc des différences notables en terme de setup avant de pouvoir travailler sur la donnée. Du point de vue l'XML, il est facile de déterminer si le service se connecte à l'image avec l'attribut *autoConnect* alors que dans le 2e cas il faut bien souvent regarder dans le code du service.

À cause de la complexité de l'organisation des données dans les composite, une dérive a conduit à utiliser directement les UID des objets au lieu des clés de composite. Le design devient alors définitivement caduque : le service a une donnée "principale", parfois difficile à déterminer et pioche ensuite des objets à sa guise dans un *pool* global, rendant ainsi le périmètre d'action d'un service encore plus difficile à lire. Sans compter que certains services mixent les clés et les UID...

1.3. Swapper
--------------

La plupart des services travaillent sur des données qui doivent être valides au lancement, ce qui est tout à fait normal, voire souhaitable. Toutefois, dans le cycle de vie d'une application, ces données ne sont pas toujours disponibles dès le lancement de l'application. À cause de l'écriture de ces applications en XML, il n'est pas possible de conditionner le lancement de services à la mise en disponibilité d'une donnée. Avec un langage de programmation fournissant des structures de contrôles, le problème ne se serait pas posé, du moins pas de façon aussi évidente.

Pour répondre à cette problématique, un service de services a été ajouté : le **SSwapper**. Il écoute l'ajout ou la suppression d'objets sur un composite et démarre ou stoppe une liste de services en fonction. En pratique, ce service est complexe à utiliser. Il dispose de plusieurs "modes" peu appréhendables sans connaissance approfondie des services; la lecture du XML est alourdie, les clés de composite ajoutées ou supprimées ne sont pas directement lisibles.

L'expérience sur plusieurs années a montré que la mise en place de ce service est toujours problématique, et pourtant elle est indispensable dans de nombreux cas.

1.4. ${GENERIC_UID}
---------------------

L'utilisation des AppConfig qui peuvent être instanciées plusieurs fois (exemple les activités) ont toujours induit l'utilisation d'une variable de substitution appelée *${GENERIC_UID}*. Cette variable est ajoutée par le développeur du XML à chaque identifiant unique de service, objet, ou canal de communication. Cette manipulation alourdit la lecture des fichiers XML. Elle est également source d'erreurs, car elle dépend du développeur qui doit vérifier que sa configuration comprend bien toutes les substitutions requises pour être instanciée plusieurs fois.

2. Propositions
====================

2.1. Nouveau bundle
----------------------
Pour intégrer les propositions suivantes tout en gardant la compatibilité avec l'ancien système, nous proposons d'introduire un nouveau bundle appXml2. Conjointement, une AppConfig2, un AppConfigManager2 sont créés pour supporter toutes les modifications liées au XML.

2.2. Un service travaille sur N données
-------------------------------------------

Nous proposons de supprimer l'indirection du composite au niveau de l'AppConfig et d'assumer le fait qu'un service travaille sur plusieurs données. Nous ne supprimons pas pour autant la donnée *::fwData::Composite* qui reste toujours utile.

Une AppConfig2 xml ne travaille donc plus sur une unique donnée. Tous les objets sont spécifiés en tête de configuration :

.. code-block :: xml

    <config>

        <object uid="modelSeries" type="::fwMedData::ModelSeries" />
        <object uid="image" type="::fwData::Image" />
        ...

Un service propose une liste d'entrées (*in*), de sorties (*out*), et d'entrées/sorties(*inout*). Cette différence dans l'accès permet de mieux identifier comment les données sont employées et de sécuriser leur utilisation (les *in* ne sont accessibles qu'en *const*). Le parsing de ces entrées/sorties est réalisé par l'AppConfig, supprimant ainsi une partie du travail répétitif de parsing et de récupération des données. Les données sont accessibles par le service à l'aide de méthodes simples comme *getInput<>(const KeyType key)*, *getOutput<>(const KeyType key)*, etc...

Au niveau du xml de l'AppConfig2, cela change également l'écriture d'un service : il n'est plus inclus dans une balise <object> et il doit donc préciser chaque donnée utilisée :

.. code-block :: xml

        <service uid="imageReader" impl="::uiIO::editor::SIOSelector">
            <out key="target" uid="image" />
            <type mode="reader" />
        </service>
            
        <service uid="mesher50ServiceUID" impl="::opVTKMesh::SVTKMesher">
            <in key="image" uid="image" autoConnect="yes" />
            <out key="modelSeries" uid="modelSeries" />
            <config>
                <percentReduction>50</percentReduction>
            </config>
        </service>

L'écriture du xml est potentiellement plus verbeuse si de nombreux services utilisent la même donnée mais localement il est plus aisé d'identifier quelles sont les données utilisées par un service. À terme, il va de soit qu'aucun service ne devra accéder à des données autrement que par ce mécanisme et l'accès par UID sera à proscrire.

Dans la continuité, une nouvelle méthode *IService::getAutoConnections()* est implémentable pour définir, pour chaque clé, les connexions à effectuer avec le service. L'attribut *autoConnect* est spécifiable globalement à toutes les données ou individuellement.

.. code-block :: cpp

    IService::KeyConnectionsMap SService::getAutoConnections() const
    {
        KeyConnectionsMap connections;
        connections.push("image", s_MODIFIED_SIG, s_UPDATE_IMAGE_SLOT);
        connections.push("image", s_BUFFER_MODIFIED_SIG, s_UPDATE_IMAGE_SLOT);
        connections.push("modelSeries", s_MODIFIED_SIG, s_UPDATE_MESH_SLOT);

        return connections;
    }
    

2.3. Données différées
------------------------

Nous proposons de supprimer le service SSwapper et autres services similaires comme le service SField. 

Pour répondre au besoin de lancement différé de services en fonction de la disponibilité d'une donnée, nous introduisons la notion de **donnée différée**. Il s'agit d'un attribut de build de la donnée, qui indique en premier lieu à l'AppConfig que la donnée ne doit pas être créée au lancement de celle-ci. Deuxièmement, cela indique implicitement que tous les services utilisant cette donnée **en entrée** ne seront pas créés eux aussi; toutefois ils seront instanciés puis démarrés automatiquement lorsque toutes leurs données différées seront disponibles.

Dans l'exemple suivant, le service *updaterReconst* travaille sur la donnée différée *reconst* **en sortie** et il est donc capable de démarrer. En revanche *organMaterial* ne peut démarrer car il utilise cette même donnée **en entrée**. Le service *updaterReconst* va produire la donnée à un instant donné, rendant la donnée *reconst* disponible pour le reste de l'AppConfig2. Le service *organMaterial* sera alors automatiquement lancé par l'AppConfig2. Si la donnée vient à disparaître il sera automatiquement stoppé. 

.. code-block :: xml
    
    <object uid="reconst" type="::fwData::Reconstruction" src="deferred" />

    <service uid="updaterReconst" impl="::ctrlSelection::updater::SObjFromSlot">
        <out key="target" uid="reconst" />
    </service>
            
    <service uid="organMaterial" impl="::uiReconstruction::organMtlEditor">
        <inout key="reconstruction" uid="reconst" />
    </service>
    
Une donnée différée est créée, supprimée ou modifiée par un service travaillant sur cette donnée **en sortie**. Pour rendre cette donnée disponible aux autres services, la méthode *::IService::registerOutput* est utilisée. Celle-ci envoie un signal qui est intercepté par l'AppConfig2, qui peut démarrer ensuite les services concernés si toutes les conditions sont remplies.
    
La fonctionnalité proposée par le SSwapper est donc toujours présent, mais intégrée à l'AppConfig2, d'une manière proche des scènes génériques VTK, Ogre ou 2D. Dans le futur, il serait souhaitable d'homogénéiser ce comportement avec du code commun. Celui-ci pourrait également être utilisé si nécessaire dans le cadre de l'écriture d'une application sans le XML, en C++ ou un autre langage pour ne pas avoir à gérer cette problématique manuellement.
          
2.4. Données optionnelles
----------------------------

Dans un certain nombre de cas, il est souhaitable qu'un service travaillant sur une ou plusieurs données différées **en entrée** ne bloque pas sa création et son lancement sur leur disponibilité. Une scène générique par exemple peut travailler sur un mesh qui n'est pas présent au lancement; toutefois elle sait gérer son absence et son apparition/disparition. Il faut donc permettre ce cas, ce qu'il est possible de faire en précisant l'attribut "optional" :

.. code-block :: xml

    <service uid="organMaterial" impl="::uiReconstruction::organMtlEditor">
        <inout key="reconstruction" uid="reconst" optional="yes"/>
    </service>
    
Dans ce contexte, un service pourra être notifié de l'apparition, de la modification ou de la disparition d'un objet grâce à la nouvelle méthode *IService::swapping(const KeyType&)*.
    
2.5. Connexions
-------------------

Pour simplifier l'écriture du xml, nous avons choisi de fusionner les balises *<connect>* et *<proxy>* dans le cadre de l'AppConfig2. La balise *<proxy>* est supprimée tandis qu'il a été ajouté la possibilité d'ajouter plusieurs signaux et un nom de canal sur la balise *<connect>*. En terme d'implémentation nous n'avons donc gardé, en réalité, que les proxys qui sont exposés dans le xml via la balise *<connect>*.

Ainsi le code suivant:

.. code-block :: xml

    <connect>
        <signal>listOrganEditor/reconstructionSelected</signal>
        <slot>updaterReconst/addOrSwap</slot>
    </connect>

    <proxy channel="modelSeriesNormalChannel">
        <signal>representationEditor/normalsModeModified</signal>
        <signal>representationEditor2/normalsModeModified</signal>
    </proxy>
            
devient :

.. code-block :: xml

    <connect>
        <signal>listOrganEditor/reconstructionSelected</signal>
        <slot>updaterReconst/addOrSwap</slot>
    </connect>

    <connect channel="modelSeriesNormalChannel">
        <signal>representationEditor/normalsModeModified</signal>
        <signal>representationEditor2/normalsModeModified</signal>
    </connect>
            

Dans le cadre d'un service utilisant une donnée différée, il faut noter que ces connexions ne sont créées/détruites que lorsque ce service est démarré/stoppé par l'AppConfig2.

2.6. Enregistrement des services
--------------------------------------

Jusqu'à présent, chaque service utilise une macro pour s'enregistrer dans une factory. Cet enregistrement est utile pour trois fonctions:

1. Instantiation du service par type,
2. Vérification de l'objet associé à un service lors de l'association effective au sein de l'OSR,
3. Listing des services (en filtrant ou non par interface de base), qui travaillent sur un type de donnée en particulier; par exemple obtenir les ::io::Reader travaillant sur des ::fwData::Image.

Plus précisément, ceci se réalise par exemple de la façon suivante :

.. code-block :: cpp

    fwServicesRegisterMacro( ::io::IReader, ::ioVTK::SReader, ::fwData::Image );

Nous nous sommes donc légitimement posé la question du devenir de cette macro avec l'avènement des données multiples sur un service. Faut-il enregistrer le type de toutes les données ? Faut-il la supprimer ? Pour l'instant, nous avons choisi le status quo. Voici pourquoi.

Pour la fonction n°1, nous n'avons pas besoin de modifier quoique ce soit. La macro pourrait même ne pas définir le type d'objet à associer, cela ne changerait rien.

Pour la fonction n°2, si nous n'enregistrons pas le type de chaque donnée, nous perdons la vérification qui se fait lors de l'enregistrement du service dans l'OSR, juste après sa création. C'est ce qui se passait jusque là... avec toutefois un gros bémol, puisque ce n'était uniquement le cas que pour les services travaillant sur une seule donnée ! Pour tous les objets travaillant sur plusieurs données, que ce soit en utilisant les clés d'un *::fwData::Composite*, ou directement en passant par les UID, cette vérification n'était pas faite à cet instant. En revanche, lors de l'utilisation de la donnée, au *start()*, à l'*update()* ou dans un slot, un *dynamic_cast()* était obligatoire et vérifiait donc finalement le type de la donnée. Donc au final si nous n'enregistrons pas le type de chaque donnée, nous retardons simplement le moment où une erreur potentielle de type est levée pour les services travaillant sur une donnée unique. Pour les services travaillant sur plusieurs données, cela ne change rien, l'erreur ne sera remontée qu'au moment de leur utilisation.

Pour la fonction n°3, en ne modifiant rien, nous gardons le comportement intact. Il est toujours possible de lister tous les *IReader* par exemple. Enregistrer tous les types de données d'un service n'aurait pas vraiment de sens pour cette fonction. Quel est l'intérêt de récupérer tous les *::arServices::ISimulator* travaillant sur une *::fwData::Image*, sachant que parmi ces services, l'un va travailler également sur un *::fwData::Mesh* et deux *::fwMedData::ModelSeries*, l'autre sur deux autres *::fwData::Image* et un *::fwData::Composite*, etc... ? Cela n'apporterait aucune information exploitable. En réalité cela n'a de sens que si le service travaille sur un seul type de donnée; donc en gardant la macro telle quelle nous remplissons toujours cette fonction.

La seule évolution envisageable serait éventuellement de séparer la fonction n°1 et la fonction n°3 en deux macros distinctes. Pour la fonction n°2, nous évaluerons à l'usage s'il est problématique ou non de remonter les erreurs tardivement.

2.7. Suppression des *${GENERIC_UID}*
---------------------------------------

Il n'est plus nécessaire d'utiliser les *${GENERIC_UID}*. L'AppConfig2 se charge lui-même des substitutions, en reconnaissant les tags XML qui désigne des identifiants uniques. 

Le seul désavantage à l'heure actuelle est la nécessité de différencier les identifiants et les simples chaînes de caratères lors du remplacement de paramètres pour les lanceurs de configuration. Cela revient à spécifier un tag **uid** au lieu de **by**. En clair, s'il s'agit d'un nom de canal de connexion ou d'un wid (le fameux *WID_PARENT*), alors vous devez mettre **uid**, sinon ce sera **by**. Quant aux identifiants d'objets, ils sont de toute façon traités différemment comme nous allons le voir dans la section suivante.

2.8. Utilisation des objets pour les lanceurs de configuration
----------------------------------------------------------------

Les services *::gui::action::SConfigLauncher* et *::fwServices::SConfigController* référencent désormais les objets qu'ils vont remplacer, au lieu de simplement effectuer une substitution. Cela est nécessaire pour les objets deferred, car l'identifiant de ces objets n'est pas connu en dehors de l'AppConfig courante (c'était déjà le cas auparavant, on passait par un path *camp* pour récupérer la clé de composite). Ceci permet également de bénéficier du start automatique avec ces objets. 

Au niveau de la configuration, là où l'on utilisait une ligne de substitution d'un identifiant d'objet, il faut une ligne de clé d'objet en *inout* (tous les paramètres d'objets seront effectivement considérés en lecture/écriture car on ne peut pas le déterminer à priori, et cela n'a pas vraiment de sens pour la substitution). Le nom de la clé d'objet correspond au nom de la variable à substituer dans la configuration cible. Ainsi le code précédent :

.. code-block :: xml

    <service uid="configLauncher" impl="::gui::action::SConfigLauncher">
        <config>
            <appConfig id="configuration">
                <parameters>
                    <parameter replace="ICON_PATH" by="${ICON_PATH}" />
                    <parameter replace="orientation" by="frontal" />
                    <parameter replace="object" by="object1" />
                    <parameter replace="channel" by="channel1" />
                </parameters>
            </appConfig>
        </config>
    </service>

Se transforme en :

.. code-block :: xml

    <service uid="configLauncher" impl="::gui::action::SConfigLauncher">
        <appConfig id="configuration" />
        <inout key="object" uid="object1" />
        <parameter replace="ICON_PATH" by="${ICON_PATH}" />
        <parameter replace="orientation" by="frontal" />
        <parameter replace="channel" uid="channel1" />
    </service>


2.9. Debug
------------

Pour aider au débogage du démarrage des services, des logs ont été ajouté au niveau INFO, indiquant par exemple qu'un service n'a pas été démarré car une ou plusieurs ne sont pas disponibles (en précisant lesquelles), ou encore qu'un service a été démarré/stoppé à cause d'une création/destruction de donnée.

De façon générale, les erreurs sont remontées de façon plus explicite en essayant de préciser un contexte, notamment l'identifiant de la configuration en particulier, pour aider à comprendre les erreurs sans avoir à lancer un débogueur.

2.10. Versions
----------------

**AppXml2** est une évolution majeure sur *Sight 11.0*. 
La compatibilité avec **appXml** restera assurée tout au long du cyle sur *Sight 11*. 
Nous prévoyons de supprimer appXml à partir de la branche *Sight 12.0*.

3. Guide de migration
===========================

Nous présentons dans la suite un ensemble de règles à appliquer pour migrer une application et/ou des activités.

3.1 Comment créer une appConfig utilisant appXml2 ?
--------------------------------------------------------

Tout d'abord dans le **Properties.cmake**, il faut remplacer les occurrences de appXml par appXml2. Par exemple :

.. code-block :: cmake

    set( NAME Application )
    set( VERSION 0.1 )
    set( TYPE APP )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        dataReg
        ...
        fwlauncher
        appXml
    )

    bundleParam(appXml PARAM_LIST config PARAM_VALUES ApplicationConfig)

devient:

.. code-block :: cmake

    set( NAME Application )
    set( VERSION 0.1 )
    set( TYPE APP )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        dataReg
        ...
        fwlauncher
        appXml2
    )

    bundleParam(appXml2 PARAM_LIST config PARAM_VALUES ApplicationConfig)

Ensuite dans le **plugin.xml**, quand vous déclarez le point d'extension de la configuration XML il faut simplement modifier *AppConfig* par *AppConfig2* :

.. code-block :: xml

    <extension implements="::fwServices::registry::AppConfig">
        <id>ApplicationConfig</id>
        <config>
        ...

en :

.. code-block :: xml

    <extension implements="::fwServices::registry::AppConfig2">
        <id>ApplicationConfig</id>
        <config>
        ...

Notez évidemment que appXml2 et AppConfig2 seront renommés en appXml et AppConfig sur la branche 0.12, après la suppression de l'actuel appXml.

3.2 Comment déclarer les objets et les services ?
--------------------------------------------------

Auparavant la configuration XML d'un appConfig contenait une unique balise object, qui comme nous l'avons noté au début de ce document, désignait la plupart du temps un composite. Suivaient imbriqués dans cette balise, les services, puis les clés du composite, avec d'éventuels Composite et donc à nouveau des services à l'intérieur, etc... Par exemple :

.. code-block :: xml

    <config>
        <object uid="root" type="::fwData::Composite">

            <service uid="srv1" impl="::namespace::SServiceImpl" />
            <service uid="srv2" impl="::generator::SMesh" >
                <config>
                    <inputImageKey>imageKey</inputImageKey>
                    <outputMesh>mesh</outputMesh>
                </config>
            </service>

            <item key="subCompositeKey">
                <object uid="subComposite" type="::fwData::Composite">

                    <item key="meshKey">
                        <object uid="mesh" type="::fwData::Mesh" >
                            <service uid="meshSrv" ... />
                        </object>
                    </item>

                </object>
            </item>

            <item key="imageKey">
                <object uid="image" type="::fwData::Image" >
                    <service uid="imageSrv" ... />
                </object>
            </item>

            <start uid="srv1" />
            <start uid="srv2" />
            <start uid="meshSrv" />
            <start uid="imageSrv" />

        </object>
    </config>

Dans cet exemple, vous pouvez remarquez la mauvaise pratique dans le service *srv2*, qui mixe l'utilisation d'une clé et d'un UID. Pour pouvoir n'utiliser que des clés dans ce service, il aurait fallu faire une référence dans le composite "root", ce qui alourdit très rapidement le fichier si cela est répété plusieurs fois.

Avec AppConfig2, tout est mis à plat, fini le décodage des imbrications. Une configuration travaille sur un ensemble d'objets puis un ensemble de services; ceux-ci vont ensuite déclarer chacun quelles données ils utilisent. L'exemple précédent devient ainsi :

.. code-block :: xml

    <config>
        <object uid="mesh" type="::fwData::Mesh" />
        <object uid="image" type="::fwData::Image" />

        <service uid="srv1" type="::namespace::SServiceImpl" />

        <service uid="srv2" type="::generator::SMesh" >
            <in key="inputImage" uid="image" />
            <inout key="outputMesh" uid="mesh" />
        </service>

        <service uid="meshSrv" ... >
            <in key="skin" uid="mesh" />
        </service>

        <service uid="imageSrv" ... >
            <in key="scan" uid="mesh" />
        </service>

        <start uid="srv1" />
        <start uid="srv2" />
        <start uid="meshSrv" />
        <start uid="imageSrv" />

    </config>

Objectivement, vous pouvez observer que le résultat est plus concis. Les deux *composites* utilitaires qui servaient juste à contenir les vraies données ont disparus. Et nous ne les regretterons pas. Tous les objets sont regroupés, suivis des services; il n'est ainsi plus nécessaire de chercher les services au milieu des items des *composites*.

Chaque service référence les données qu'il utilise avec un identifiant unique, que nous nommons simplement par l'attribut *id*. Il s'agit de l'identifiant de la donnée dans la configuration XML courante. Il n'y a plus d'alternative comme auparavant. Toutefois, pour l'instant il est toujours possible d'utiliser directement l'UID de l'objet mais cela sera proscrit dans le futur. Le service utilise une clé, autrement dit un alias, pour désigner cette donnée dans son code. L'ajout de cette clé, si tant est bien sûr qu'elle possède un nom intelligible, permet également de mieux comprendre l'utilisation qui est faite de la donnée, même dans le cas d'une donnée unique. L'ajout des types d'accès (*in*, *inout*, *out*) aident également à mieux comprendre le rôle rempli par chacune des données. 

Enfin de façon plus générale, n'oubliez pas de ne plus utiliser de *${GENERIC_UID}* et de remplacer les tags **by** par **uid** dans les remplacements des paramètres de lancement de configuration. Par ailleurs les services ne se déclarent plus en précisant **type** et **impl**. Seul **type** est précisé, mais il correspond à la vraie classe devant être instanciée, pour être plus cohérent avec la déclaration des objets. Autrement dit, ce qui était dans **impl** doit être copié dans **type** et ensuite **impl** doit être supprimé.


3.3 Comment choisir entre Input, InOut et Output ?
----------------------------------------------------

Pour convertir vos services ou en écrire de nouveaux, il vous faut déterminer le type d'accesseur pour chaque donnée. 

1. Lecture seule
_________________

Pour les données accessibles en lecture seule, c'est simple, il faut prendre *in*. Toutefois dans le cadre de la migration, il se peut que la conversion soit difficile à cause de l'apparition du *const*. Si c'est possible, faites les modifications nécessaires, dans le cas contraire vous devrez prendre InOut temporairement et prendre note de changer cela plus tard.

2. Ecriture seule
__________________

Il est important de comprendre que les out sont des données qui vont être **produites**. Ce sont donc nécessairement des données *deferred* dans l'AppConfig. Là où l'on pourrait penser par exemple que nos lecteurs accèdent à leur donnée en *out*, en fait ce n'est pas le cas. Ceux-ci travaillent en effet sur une donnée qui est déjà allouée et ils ne font que la modifier. En revanche les services qui extraient des objets au sein de *composites* ou des fields utilisent des vraies *out*.

3. Modification
_________________

Si vous êtes ni dans le premier, ni dans le deuxième cas, alors nécessairement vous êtes en *inout*.

3.4 Comment accéder aux objets d'un service ?
-----------------------------------------------

Pour accéder aux données d'un service, il existe trois nouvelles méthodes différentes pour récupérer respectivement une entrée (*in*), une entrée/sortie(*inout*) ou une sortie (*out*):

.. code-block :: cpp

    template<class DATATYPE> CSPTR(DATATYPE) getInput( const KeyType &key) const;
    template<class DATATYPE>  SPTR(DATATYPE) getInOut( const KeyType &key) const;

Notez bien que *getInput()* renvoie un pointeur intelligent **const**. Et oui la fête est finie, on arrête de faire n'importe quoi avec n'importe qui ! Et ce n'est qu'un début, d'autres améliorations viendront plus tard, nous l'espérons, pour éviter les problèmes d'accès concurrentiels.

Pour éviter de modifier tous les services actuels, les anciennes méthodes fonctionnent toujours, mais avec un comportement adapté aux changements :

.. code-block :: cpp

    ::fwData::Object::sptr getObject();
    template< class DATATYPE > SPTR(DATATYPE) getObject();

Si un service utilise plusieurs données, alors *getObject()* renverra simplement le premier objet déclaré dans la liste des données du service dans l'XML. Si un service ne travaille sur aucune donnée (les *::gui::view::IView* par exemple) alors *getObject()* renverra un objet **dummy** de type *::fwData::Composite* créé spécialement par l'AppConfig courante.

Règle de codage
________________

Si vous réutilisez plus d'une fois un nom de clé d'objet, alors il est recommandé d'utiliser une *string* en *static const* dans votre fichier source. Il n'est pas utile de le mettre en membre statique de classe puisqu'il ne sera normalement pas utilisé à l'extérieur de la classe.

3.5 Comment documenter les objets d'un service ?
--------------------------------------------------

Le guide de style préconise de documenter les données d'un service dans la section XML de la doxygen du service, au-dessus de la classe. Jusqu'à trois sous-sections doivent être ajoutées pour chaque catégorie de données, et pour chaque donnée, le nom de la clé et le type de la donnée (entre crochets) doivent être décrits.

    .. code-block:: cpp

         *
         * @section XML XML Configuration
         *
         * @code{.xml}
                <service impl="::namespace::SService">
                    <in key="data1" uid="model" />
                    <inout key="data2" uid="mesh" />
                    <out key="data3" uid="image2" />
                    <out key="data4" uid="image1" />
                    <option1>12</option1>
                    <option2>12</option2>
                </service>
           @endcode
         * @subsection Input Input
         * - \b data1 [::fwMedData::ModelSeries]: blablabla.
         * @subsection In-Out In-Out
         * - \b data2 [::fwData::Mesh]: blablabla.
         * @subsection Output Output
         * - \b data3 [::fwData::Image]: blablabla.
         * - \b data4 [::fwData::Image]: blablabla.
         * @subsection Configuration Configuration
         * - \b option1 : first option.
         * - \b option2(optional) : second option.
         */

3.6 Comment gérer un nombre indéterminé d'objets dans un service ?
--------------------------------------------------------------------

Il se peut que vous ayiez besoin d'avoir une liste d'objets de même type en entrée. Dans ce cas, la solution pourrait être de définir en entrée toutes les clés : 

.. code-block :: xml

        <service uid="srv" ... >
            <in key="matrix1" uid="matrixFromTag21" />
            <in key="matrix2" uid="matrixFromTag103" />
            <in key="matrix3" uid="matrixFromTag104" />
            ...
            <in key="matrixN" uid="matrixFromTag3" />
        </service>

Toutefois récupérer les objets dans le service s'avèrerait fastidieux car le nombre de clés n'est pas connu à l'avance et il faudrait donc "tester" l'existence ou non des clés.

Pour éviter cette gestion fastidieuse, vous pouvez utilisez la fonctionnalité des groupes de clés :

.. code-block :: xml

        <service uid="srv" ... >
            <in group="matrix" />
                <key uid="matrixFromTag21" />
                <key uid="matrixFromTag103" />
                <key uid="matrixFromTag104" />
                ...
                <key uid="matrixFromTag3" />
            </in>
        </service>

Ces objets peuvent ensuite être récupérés dans le cpp à l'aide des fonctions :

.. code-block :: cpp

    template<class DATATYPE> CSPTR(DATATYPE) getInput( const KeyType &keybase, 
                                                       size_t index) const;
    template<class DATATYPE>  SPTR(DATATYPE) getInOut( const KeyType &keybase, 
                                                       size_t index) const;
    size_t getKeyGroupSize(const KeyType &keybase) const;

Par exemple:

.. code-block :: cpp

    ::fwData::Object::csptr obj1 = this->getInput< ::fwData::Object>("group", 1);
    ::fwData::Object::csptr obj2 = this->getInput< ::fwData::Object>("group2");

    const size_t groupSize = this->getKeyGroupSize("group");
    for(int i = 0; i < groupSize; ++i)
    {
        auto obj = this->getInput< ::fwData::Object>("group", i);
        ...
    }


3.7 Dois-je modifier le code de mon service ?
-------------------------------------------------

Mon service ne fonctionne pas
________________________________

Il y a plusieurs raisons pour lesquelles votre service pourrait ne plus fonctionner. La plus courante concerne le cas où le service utilise plusieurs données avec des clés de composite. Le composite ayant disparu, ça se passera forcément mal. 

Dans ce cas ou dans un autre la première question à vous poser est la suivante : est-ce que mon service est vraiment utile ? Dans la négative, c'est simple supprimez-le. Dans l'affirmative, dommage, vous n'avez pas le choix, oui vous devez faire des modifications. 

Mon service fonctionne
__________________________

C'est à vous de voir. Si vous utilisez un seul objet et que tout fonctionne bien, rien ne vous oblige à changer quoique ce soit. Tant que nous devons assurer la compatibilité avec appXml, il vaut mieux même éviter. Toutefois, si votre service est utilisé **uniquement** dans des applications appXml2 et/ou si votre service utilise plusieurs données avec des UID, alors vous pouvez envisager de migrer le service sur la nouvelle API, le passage vers 0.12 en sera facilité, et cela vous permettra sans doute d'avoir un code de gestion des données simplifié.

Que faire ?
_______________

1. Récupération d'un pointeur sur une donnée
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Le parsing des données n'est plus nécessaire, servez-vous en à bon escient. Exemple classique :

.. code-block :: xml

    <service uid="..." type="...">
        <imageKey>CTImage</imageKey>
    </service>

.. code-block :: cpp

    void SService::configuring()
    {
        ConfigType cfg = m_configuration->findConfigurationElement("imageKey");
        SLM_ASSERT("Missing element 'imageKey'", cfg );

        m_imageKey = cfg->getValue();
        SLM_ASSERT("Missing 'imageKey' data", !m_imageKey.empty());
    }

    void SService::starting()
    {
        ::fwData::Composite::sptr comp = this->getObject< ::fwData::Composite>();
        ::fwData::Image::ptr image = comp->at("image");
    }

Dans appXml2, cela se réduit en (à ajuster évidemment pour l'accesseur) :

.. code-block :: xml

    <service uid="..." type="...">
        <inout key="image" uid="CTImage" />
    </service>

.. code-block :: cpp

    void SService::starting()
    {
        ::fwData::Image::ptr image = this->getInput< ::fwData::Image>("image");
    }

2. Connexions pour N données
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Auparavant, il était nécessaire de connecter manuellement chaque donnée. 

.. code-block :: xml

    <service uid="..." type="...">
        <imageKey>CTImage</imageKey>
        <meshUID>segmentation</meshUID>
    </service>

.. code-block :: cpp

    void SService::SService()
    {
        m_connections = ::fwServices::helper::SigSlotConnection::New();
    }

    void SService::starting()
    {
        ::fwData::Composite::sptr comp = this->getObject< ::fwData::Composite>();
        ::fwData::Image::ptr image = comp->at("image");

        auto obj = ::fwTools::fwID::getObject(m_objectUid);
        ::fwData::Mesh::sptr mesh = ::fwData::Object::dynamicCast(obj);

        m_connections->connect(image, s_BUFFER_MODIFIED_SIG, 
                               this->getSptr(), s_UPDATE_IMAGE_SLOT);
        m_connections->connect(mesh, s_VERTEX_MODIFIED_SIG_SIG, 
                               this->getSptr(), s_UPDATE_VERTEX_SLOT);
    }

    void SService::stopping()
    {
        m_connections->disconnect();
    }

Avec appXml2, il est fortement recommandé d'utiliser la nouvelle méthode *getAutoConnections()* qui repose sur l'attribut *autoConnect*. Cela donne encore plus d'indices dans le XML, sur les interactions du service avec ses données. Notez par ailleurs que l'attribut *autoConnect* peut-être spécifié globalement au niveau du service pour s'appliquer à toutes les données :

.. code-block :: xml

    <service uid="..." type="...">
        <inout key="image" uid="CTImage" autoConnect="yes" />
        <inout key="mesh" uid="segmentation" autoConnect="yes" />
    </service>

.. code-block :: cpp

    ::fwServices::IService::KeyConnectionsMap SService::getAutoConnections() const
    {
        KeyConnectionsMap connections;
        connections.push( "image", ::fwData::Image::s_BUFFER_MODIFIED_SIG, 
                          s_UPDATE_IMAGE_SLOT );
        connections.push( "mesh", ::fwData::Mesh::s_VERTEX_MODIFIED_SIG,
                          s_UPDATE_VERTEX_SLOT );
        return connections;
    }

N'hésitez pas à utiliser des static const string pour stocker le nom des clés et surtout documentez ce qui doit être connecté ou non dans la doxygen du service. 

3. Gérer la compatibilité
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si votre service est utilisé dans des applications appXml et appXml2 (cas peu probable en dehors de nos dépôts internes) alors il faut faire attention et faire les modifications en vérifiant bien de ne pas casser son utilisation en appXml. Pour vous aider, il existe la méthode suivante qui vous permettre d'écrire du code spécifique :

.. code-block :: cpp

    static bool ::fwServices::IService::isVersion2();

Cherchez ses utilisations dans le code et vous trouverez des exemples d'utilisation pour le parsing par exemple.



3.8 Ciel un swapper ! 
---------------------------

Ok les choses sérieuses commencent... Pour illustrer la migration d'une configuration comprenant un swapper, prenons le cas du **Tuto09MesherWithGenericScene** (certains identifiants ou types ont été raccourcis pour que le code ne déborde pas de la page) :

.. code-block :: xml

    <service uid="updaterReconst" impl="::ctrlSelection::updater::SObjFromSlot">
        <compositeKey>reconstruction</compositeKey>
    </service>

    <service uid="mgr" impl="...::SwapperSrv" autoConnect="yes">
        <mode type="stop" />
        <config>
            <object uid="reconstruction" type="::fwData::Reconstruction">
                <service uid="organMtlEditor" impl="...::organMtlEditor"/>
                <service uid="repEditor" impl="...::RepresentationEditor"/>

                <connect>
                    <signal>repEditor/normalsModeModified</signal>
                    <slot>modelSeriesAdaptorUid/updateNormalMode</slot>
                </connect>
            </object>
        </config>
    </service>

    <item key="modelSeries">
        <object uid="modelSeriesUID" type="::fwMedData::ModelSeries">
            <service uid="listOrganEditor" impl="..::SModelSeriesList">
                <columns>
                    <organ_name>@organ_name</organ_name>
                </columns>
            </service>
        </object>
    </item>

    <connect>
        <signal>listOrganEditor/reconstructionSelected</signal>
        <slot>updaterReconst/addOrSwap</slot>
    </connect>

    <start uid="updaterReconst" />
    <start uid="mgr" />

Dans cet exemple, on souhaite *simplement* afficher les éditeurs de matériau *organMtlEditor* et de reconstruction *repEditor* pour la sélection courante. Le service *listOrganEditor* signale le service *updaterReconst* qui ajoute une clé nommée *reconstruction* dans le composite. Cet objet n'aura jamais d'UID utilisable dans ce XML. Pour pouvoir utiliser cette donnée, le seul moyen est donc de recourir à un **SSwapper**, qui va démarrer les services d'édition quand la clé est ajoutée, et les supprimera quand la clé est supprimée.

Or avec AppXml2, et c'est en grande partie ce pour quoi il a été conçu, ce comportement de démarrage et d'arrêt automatique de services est intégré à l'AppConfig et ne nécessite pas de service utilitaire comme SSwapper. Ce mécanisme repose sur l'utilisation de **donnée à création différée**. Jusqu'à présent, lorsque vous déclarez une donnée dans appXml2 (ou appXml), celle-ci est créée par l'AppConfig (sauf si vous avez spécifié *src="ref"*). Le principe de la donnée différée, c'est simplement d'indiquer à l'AppConfig qu'elle ne doit pas créer la donnée, car celle-ci sera produite par un service.

Ainsi le cas du **Tuto09MesherWithGenericScene** se simplifie de la façon suivante :

.. code-block :: xml

    <object uid=" reconstUid" type="::fwData::Reconstruction" src="deferred"/>

    <service uid="listOrganEditor" impl="..::SModelSeriesList">
        <in key="modelSeries" uid="modelSeriesId" />
        <columns>
            <organ_name>@organ_name</organ_name>
        </columns>
    </service>

    <service uid="updaterReconst" impl="::ctrlSelection::updater::SObjFromSlot">
        <out key="object" uid=" reconstUid" />
    </service>

    <service uid="organMtlEditor" impl="::uiReconstruction::organMtlEditor">
        <inout key="reconstruction" uid=" reconstUid" />
    </service>

    <service uid="repEditor" impl="::uiReconstruction::RepresentationEditor">
        <inout key="reconstruction" uid=" reconstUid" />
    </service>

    <connect>
        <signal>listOrganEditor/reconstructionSelected</signal>
        <slot>updater reconstUid/addOrSwap</slot>
    </connect>

    <connect>
        <signal>repEditor/normalsModeModified</signal>
        <slot>modelSeriesAdaptorId/updateNormalMode</slot>
    </connect>

Il faut donc commencer par déclarer la donnée reconstruction avec l'attribut *src="deferred"*. Les deux éditeurs sont extraits du *SSwapper* qui a disparu. Ensuite on indique à ces deux éditeurs qu'ils travaillent sur cette donnée... et c'est terminé ! Ils seront démarrés, leurs signaux/slots connectés lorsque *updater reconstUid* créera la donnée du point de vue de l'AppConfig. Ce service utilise en effet la reconstruction **en sortie**, il n'a donc pas besoin de la donnée pour démarrer puisqu'il indique ainsi que c'est lui qui va la produire. 

Par exemple, *SObjFromSlot* enregistre la donnée de la manière suivante :

.. code-block :: cpp

    this->setOutput("object", objectSptr);

L'AppConfig est signalée et déclenche alors les actions en conséquence.

3.9 Les données optionnelles
------------------------------

Dans l'exemple précédent, nous avons vu qu'une donnée en **out** différée n'empêchait pas le service de démarrer. Il est possible d'avoir ce comportement également sur les données en **in** et **inout** en précisant dans l'XML qu'elles sont optionnelles :

.. code-block :: cpp

    <service uid="organMtlEditor" impl="::uiReconstruction::organMtlEditor">
        <inout key="reconstruction" uid=" reconstUid" optional="yes"/>
    </service>

Pour être notifié de l'arrivée de la donnée, vous pouvez utiliser *IService::swapping(const KeyType&)*. Toutefois cela complique forcément la gestion des données, et si c'est possible, il est plutôt recommandé d'écrire des services ne travaillant que sur des données présentes. Actuellement, les données optionnelles sont utilisées pour les services qui agissent comme des managers de service comme *::fwRenderVTK::SRender*, *::scene2D:Render*, etc...

3.10 Les lanceurs de configuration
----------------------------------

Si vous utilisez un service *SConfigLauncher* ou *SConfigController*, alors vous devez modifier leur configuration comme indiqué aux sections 2.7 et 2.8.


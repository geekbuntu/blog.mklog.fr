Title: JS et Architecture, quelques pistes et conseils...
Author: Mickael Daniel
Date: Feb 05 2011 14:00:00 GMT+0100 (CDT)
Categories: Javascript

Lorsque que l'on travaille sur un site ou une application web de taille conséquente, vous pouvez vite vous retrouver avec une quantité non négligeable de Javascript. Situation dans laquelle il devient nécessaire de penser en terme d'organisation de code, design patterns et architecture.

Dans cet article, nous ferons un rapide tour des différentes (très bonnes) ressources que l'on peut trouver sur le sujet. Ces ressources et les principes qu'elles essaient d'expliquer représente une base robuste et solide à même d'évoluer en fonction de vos besoins techniques et fonctionnels.

Parce qu'une architecture bien pensée et et correctement implémentée est une composante essentielle de la maintenance de votre application. Elle permet l'intervention de larges équipes en fournissant un socle solide et modulaire, une voie à suivre claire et bien définie.

L'architecture d'applications web modernes (nous resterons partie Front, ces principes peuvent être appliqués indépendamment du langage coté backend mis en oeuvre) est un vaste sujet qui ne saurait être résumé à la taille d'un post, nous ne verrons ici que certains points / conseils qui pourront réellement vous sauver la vie (surtout si votre application est ammenée à prendre de l'ampleur).

Ce post parlera plus particulièrement de jQuery étant donné qu'il représente aujourd'hui le fwk le plus utilisé (et qu'il représente peut-être celui nécessitant le plus de travail de réflexion lorsque l'on se heurte au problème d'organisation de code) mais ces principes peuvent prendre sens avec tout autre toolkit/fwk.

Qui ne s'est jamais retrouvé devant un document.ready interminable, faisant tout et n'importe quoi et qui au fur et à mesure de l'avancement du projet devient tout sauf propre et maintenable? Parce qu'une approche centré sur le DOM avec des chaînes jQuery interminables ne suffit plus. Il est facile de produire du code médiocre avec jQuery mais il n'est pas impossible de garder les chôses "clean".


## Diminuer le couplâge de nos composants

### Pubsub Screencast - Rebecca Murphey
[Pubsub Screencast](http://blog.rebeccamurphey.com/pubsub-screencast)

Superbe screencast des concepts de custom events et système pub/sub par Rebecca Murphey. L'approche pub/sub est vraiment une réponse des plus adaptée dès lors que l'on travaille sur des applis de moyenne à grande envergure. Cette présentation explique exactement ce que l'on essaiera d'éviter et comment mettre en oeuvre une approche basée sur les évenements pour assurer la communication de nos composants.

<iframe src="http://player.vimeo.com/video/16234207" width="700" height="394" frameborder="0"></iframe><p><a href="http://vimeo.com/16234207">Rebecca on Pubsub</a> from <a href="http://vimeo.com/yayquery">yayQuery</a> on <a href="http://vimeo.com">Vimeo</a>.</p>

### Functionality Focused Code Organization - Rebecca Murphey

<div style="width:425px; margin: auto;" id="__ss_5462869"><strong style="display:block;margin:12px 0 4px"><a href="http://www.slideshare.net/rmurphey/functionality-basedorg" title="Functionality Focused Code Organization">Functionality Focused Code Organization</a></strong><object id="__sse5462869" width="425" height="355"><param name="movie" value="http://static.slidesharecdn.com/swf/ssplayer2.swf?doc=functionality-based-org-101016192130-phpapp02&stripped_title=functionality-basedorg&userName=rmurphey" /><param name="allowFullScreen" value="true"/><param name="allowScriptAccess" value="always"/><embed name="__sse5462869" src="http://static.slidesharecdn.com/swf/ssplayer2.swf?doc=functionality-based-org-101016192130-phpapp02&stripped_title=functionality-basedorg&userName=rmurphey" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="355"></embed></object><div style="padding:5px 0 12px">View more <a href="http://www.slideshare.net/">presentations</a> from <a href="http://www.slideshare.net/rmurphey">Rebecca Murphey</a>.</div></div>

Exemple d'application: [github.com/rmurphey/ffco](http://github.com/rmurphey/ffco)

Encore une superbe présentation offert à la communauté par Rebecca Murphey. Très bonne vue d'ensemble des problèmes que l'on sera amené à résoudre et approche pragmatique de ceux-ci avec un exemple concret d'application mettant en oeuvre:

#### 1. L'approche pubsub     
    // En utilisant un plugin pubsub
    $('input.magic').click(function(e) {
      // On publie un "topic" quand quelque chose arrive
      $.publish('/something/interesting', [ e.target.value ]);
    });

    // On notifie l'application de notre intérêt pour un "topic"
    $.subscribe('/something/interesting', function(val) {
      console.log(val);
    });
    
    // En utilisant bind/trigger
    (function() {
      var d = $(document); // On pourrait utiliser ici un application namespace $(myApp) où l'ensemble de notre fwk applicatif se trouve
      
      $('input.magic').click(function(e) {
        // On publie un "topic" quand quelque chose arrive
        d.trigger('/something/interesting', [ e.target.value ]);
      });
      
      d.bind('/something/interesting', function(e, val) {
        console.log(val);
      });
    })();
    
    
    
#### 2. require.def (RequireJS - [http://requirejs.org/docs/api.html#define](http://requirejs.org/docs/api.html#define))
> Définit une fonction qui sera éxecuté quand le module est inclu. La fonction peut retourner une valeur, mais n'y est pas forcé. Si des dépendances sont spécifiés, elles seront disponibles en tant qu'argument de fonction.


    // Pattern Object: Retourne un objet, bien que la fonction de définition peut 
    // clore les autres variables qui ne seront pas visibles en dehors de cette fonction
    require.def(function() {
      var privateThing = 'myPrivateThing',

          privateObj = {
            maxLength : 5,

            setPrivateThing : function(val) {
              // creating a setter lets us enforce some rules if we want
              if (val.length > this.maxLength) {
                console.log('TOO MUCH');
                return;
              }

              privateThing = val;
            },

            otherMethod : function() {
              console.log(privateThing);
            }
          };

      // return just the parts of our object we want to expose;
      // note that privateObj and privateThing aren't exposed
      // outside of the enclosing function.
      return {
        setPrivateThing : $.proxy(privateObj, 'setPrivateThing'),
        publicMethod : $.proxy(privateObj, 'otherMethod')
      };
    });
    
    // Pattern Factory: Retourne une fonction, qui, quand appellé, retourne une instance d'un objet
    // qui est définit à l'intérieur du module. Ce code retourne une factory pour créer des instances de Person.
    require.def(function(){
      var Person = {
        intro : 'My name is ',
        outro : '. You killed my father. Prepare to die.',

        speak : function() {
          console.log(
            this.intro, 
            this.firstName, 
            this.lastName,
            this.outro
          );
        }
      };

      return function(config) {
        return $.extend(Object.create(Person), {
          firstName : config.firstName,
          lastName : config.lastName
        });
      };
    });

#### 3. Pattern Mediators/Views/Services

* Mediators
> Configurent les vues et services et gère la communication entre eux, éliminant le besoin de communication directe.

* Views
> Affichent les données, observent les événements utilisateurs et propagent des messages que les mediators peuvent écouter et y réagir. Peut également fournir une API pour mettre à jour les données.

* Services
> Gèrent les données et états, exposent une API publique limité pour les mediators.

Exemple d'application: [github.com/rmurphey/ffco](http://github.com/rmurphey/ffco)

Tout à fait brillant, je conseille fortement d'y jeter un oeil.

## Exécution automatique de DOM Ready
Sur un projet, on est souvent tenté d'exécuter un ensemble d'instruction commune à notre application (une fonctionnalité présente sur la totalité ou quasi-totalité des pages de notre appli, gestion d'erreurs etc.), tout comme un ensemble d'instruction spécifique à une ressource (URL) donnée de notre appli.

Il devient alors particulièrement intéressant de disposer d'un mécanisme automatisant l'appel à un dom ready pour nous.

### Markup-based unobtrusive comprehensive DOM-ready execution - Paul Irish

[Automate firing of onload events](http://paulirish.com/2008/automate-firing-of-onload-events/)
[Markup-based unobtrusive comprehensive DOM-ready execution](http://paulirish.com/2009/markup-based-unobtrusive-comprehensive-dom-ready-execution/)

L'idée de Paul Irish permet de se baser sur le markup HTML (plus particulièrement d'une classe sur le &lt;body /&gt; tag) pour dynamiquement exécuter une méthode correspondante. Ceci permet de complètement éliminer le besoin de manuellement exécuter une fonction dom ready selon notre besoin (ou page courante). Je vous recommande vivement la lecture des deux posts de Paul.

[Extending Paul Irish’s comprehensive DOM-ready execution](http://www.viget.com/inspire/extending-paul-irishs-comprehensive-dom-ready-execution/)

Jason Garber a également publié son approche sur le blog Viget Inspire blog, inspiré par l'implémentation de Paul. Cela revient à utiliser des attributs data-* au lieu de classes CSS pour executer nos actions:

    <body data-controller="myapp" data-action="init">
    
Je suis fan.

## Retour sur la question du couplage

Habituellement et quand je dispose de la liberté d'action pour le faire, j'aime garder mes composants de pages dans leur propre "SandBox" (bac à sable). Que ce soit au niveau CSS ou JS, j'essaie de namespacer au mieux et le plus possible chacun des composants identifiés, le but étant d'être le plus modulaire possible. Niveau JS, le plus souvent l'essentiel de la logique est contenu dans des objets littérals (et namespacés) en essayant d'adopter une approche le moins centré sur le DOM possible en essayant de penser en terme d'API plutôt qu'en terme d'&lt;ul /&gt; et de &lt;li /&gt;.

Le communication entre les composants est un point critique de toute application web et, généralement, représente l'endroit où votre couplage se créera. On se retrouve vite à appeler une fonction/méthode d'un composant depuis un autre composant (BOOOM! Couplage fort,si le premier composant n'existe pas, la feature ne fonctionnera pas). Une solution permettant d'éviter sinon limiter ce phénomène est d'utiliser les custom events jQuery qui sont particuliérement adaptés pour garder nos composants découplés les uns des autres, mais avoir à utiliser le DOM peut porter à confusion (un trigger ou bind doit se faire sur une sélection jQuery). L'utilisation d'une approche pubsub permet de s'abstraire encore un peu plus du DOM et de 
simplifier la communication en utilisant un "bus" commun de message (équivalent à utiliser les custom events sur l'élement le plus haut de l'arbre DOM, en loccurence, $(document)).

### Petit apparté sur les custom events

Dans un précédent projet, l'approche custom event nous a réellement aidé dès lors que l'on était amené à gérer la communication de nos composants (page principale, un accordéon, une dialogue - avec de nombreuses possibles sur une même page - etc.). Chaque composant représente généralement une V-View dans le sens MVC du terme (Nous utilisions Struts 2 et le coupe Apache Tiles / JSP). Le principe mis en oeuvre, que nous détaillerons un peu plus en détail dans le prochain chapitre, implique que chaque composant n'a strictement aucune connaissance des autres composants de notre application. On ne peut pas (doit pas) appeller directement de fonctionnalité (fonction/méthode) ou directement manipuler d'élements DOM appartenant à un autre module. 

Si je dois mettre à jour un champ d'un autre composant (composant B) à partir d'un composant donné (composant A) par exemple, je pense en terme d'API en fournissant au composant A une méthode me permettant de mettre à jour ce champ et en utilisant les custom events (ou pubsub) pour notifier l'application que quelque chose vient d'arriver. Le composant B écoutant cet événement y répondra par la mise à jour du champ correspondant.

L'approche custom event implique d'effectuer les appels trigger/bind sur un élement particulier du DOM, l'événement se propageant (on parle de bubbling) alors jusqu'à l'élement le plus haut de l'arbre DOM à moins qu'il ne rencontre en chemin un eventHandler (gestionnaire d'événement, simplement une fonction executée quand un événement survient, click, focus, submit etc. ou customs) qui stope la propagation de l'évenement.

L'approche pubsub permet de s'abstraire de cette problématique. Ceci dit, sans pubsub, décider d'utiliser un élément commun, généralement l'élément DOM le plus haut dans l'arbre (document), revient à utiliser celui-ci comme un bus d'événements et permettra la même abstraction.

Sinon, saviez vous que trigger/bind marchait aussi sur des objets classiques? Autrement dit, n'étant pas forcément un élement de votre arbre DOM.
On peut parfaitement utiliser la syntaxe $(obj).bind('stuffGotSaved') et $(obj).trigger('stuffGotSaved'). Bien sûr, l'objet passé à bind et trigger devra être le même (généralement, on utilisera le namespace de notre application framework). Un détail je vous l'accorde mais suffisamment méconnue pour le souligner.

Si vous voulez en savoir un peu plus sur les customs events en JS:

* [API jQuery Bind](http://api.jquery.com/bind) 
* [API jQuery Trigger](http://api.jquery.com/trigger)
* [Custom events in JavaScript - Nicolas C. Zakas](http://www.nczonline.net/blog/2010/03/09/custom-events-in-javascript/)

## Scalable JavaScript Application Architecture - Nicholas C. Zakas
Enfin, nous terminerons ce billet sur une de mes présentation et vidéo préférée. Les ressources sur la conception d'architecture JS sont particulièrement rare et celle-ci fournit un début de réponse plus que solide.

<div><object width="576" height="324"><param name="movie" value="http://d.yimg.com/nl/ydn/default/player.swf"></param><param name="flashVars" value="vid=15614368&"></param><param name="allowfullscreen" value="true"></param><param name="wmode" value="transparent"></param><embed width="576" height="324" allowFullScreen="true" src="http://d.yimg.com/nl/ydn/default/player.swf" type="application/x-shockwave-flash" flashvars="vid=15614368&"></embed></object></div>

Slides: [scalable-javascript-application-architecture](http://www.slideshare.net/nzakas/scalable-javascript-application-architecture)

Quelques concepts vraiment brillant dans cette présentation. A voir et revoir.

<img class="mk-blog-img-center" src="/js-et-architecture/scallable-js-arch.jpg" alt="Base Libary / Application Code / Sandbox / Modules" title="Base Libary / Application Code / Sandbox / Modules">

L'idée principale de Zakas dans cette vidéo est de se reposer sur un socle Base Library / Application Core / Sandbox / Modules où:

* chaque module n'a aucune connaissance du reste de l'application et n'a accès qu'à lui même et à l'interface fournise par la sandbox. Le rôle d'un module est de fournir une fonctionnalité et représente souvent un ensemble cohérent de markup HTML de CSS et JS.
* la sandbox fournit une API consistente et dont le rôle peut répondre à des problématique de sécurité (détermine quelles parties du framework un module peut accéder) ou de communication (en fournissant une API permettant d'envoyer et d'écouter des messages bien que ce soit le Core qui se charge effectivement des détails du Mediator Pattern (pubsub))
* l'application core gère le cycle de vie des modules (init/destroy, active/inactive). Il peut également gérer le traitement des erreurs.
* la base library (jQuery, YUI, prototype, etc.) est utilisé par l'application core. Idéallement, seulement ce dernier à connaissance de la librairie utilisée.

Nicholas Zakas explique également [(sur ce thread JSMentors)](http://www.mail-archive.com/jsmentors@googlegroups.com/msg01279.html) que si vous prévoyez de n'utiliser qu'une librairie JS et êtes à l'aise avec cette idée, alors allez y et laissez la en dehors de l'abstraction de votre framework. Il explique, par exemple, que chez Yahoo, ils savent qu'ils ne vont certainement utiliser que YUI, donc ce n'est pas un problème si les modules accèdent directement à YUI. Si par contre, le projet était pour le compte d'une autre société, on pourrait choisir d'abstraire certaine parties communes juste au cas où l'on décide de changer de librairies JS. Le but principal étant de ne jamais avoir à ré-écrire de modules quelque soit les changements qui peuvent être apportés au reste de l'architecture. Mais si l'on est confortable à coupler notre application à une librairie donnée, on peut l'utiliser directement dans nos modules.

### Mot de la fin
Le sujet d'architecture JS est un sujet bien trop vaste pour être couvert par un simple post (d'autant plus par un simple bonhomme comme moi ;) ). Il représente néanmoins un domaine qui me passionne littéralement, raison pour lesquelles j'effectue de nombreuses recherches sur le sujet et suis très à l'écoute des avancées de la communauté dans le domaine. Ce blog et ses posts se référant au domaine d'architecture JS se contentent d'essayer de vous fournir les liens et ressources qui m'ont réellement aidé ou contribué à faire avancer ma compréhension du sujet. Suivront sûrement d'autres articles traitant des mêmes problématiques et vous présentant l'approche des plus grands et plus actifs devs. front sur le sujet (Alex Sexton, Rebecca Murphey, Paul Irish, Nicolas C. Zakas etc. pour n'en citer que queqlues uns), avec dans un premier temps une présentation 
[de l'approche d'Alex Sexton](http://alexsexton.com/?p=51) (qui fait beaucoup penser à la Widget Factory) et sa mise en oeuvre conjointe avec RequireJS (et son approche modulaire).
 
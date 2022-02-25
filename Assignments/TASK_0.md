# Se familiariser avec l'existant

## A- Exécution

Compilez et lancez le programme.

Allez dans le fichier `tower_sim.cpp` et recherchez la fonction responsable de gérer les inputs du programme.
Sur quelle touche faut-il appuyer pour ajouter un avion ? c
Comment faire pour quitter le programme ? x ou q
A quoi sert la touche 'F' ? à mettre en pleine écran

Ajoutez un avion à la simulation et attendez. 
Que est le comportement de l'avion ? Il viennent, font des tours, et repartent
Quelles informations s'affichent dans la console ? nom et actions de l'avion 

Ajoutez maintenant quatre avions d'un coup dans la simulation.
Que fait chacun des avions ? arrivée de la'avion, son "service, son départ

## B- Analyse du code

Listez les classes du programme à la racine du dossier src/. 

Aircraft, AircraftTypes, / (Avion, chacun a un AircraftTypes)
Airport, AirportType,/ (Aéroport, chacun a un type)
Point2D, Point3D,  / Coté graphique
RunWay, Terminal, / Route où l'avion roule, zone de l'aéroport
Tower, TowerSimulation, 
WayPoint,  est ce que l'avion est sur la terre ou dans les airs

Pour chacune d'entre elle, expliquez ce qu'elle représente et son rôle dans le programme.


Pour les classes `Tower`, `Aircaft`, `Airport` et `Terminal`, listez leurs fonctions-membre publiques et expliquez précisément à quoi elles servent.

Tower
    
    
    WaypointQueue get_instructions(Aircraft& aircraft);
    donnez les différentes actions de l'avion
    
    void arrived_at_terminal(const Aircraft& aircraft);
    annonce quand l'avion arrice sur le terminal


Aircraft
    /*Donne le numéro de vol*/
    const std::string& get_flight_num() const { return flight_number; }
    
    /*Donne la distance au point donné*/
    float distance_to(const Point3D& p) const { return pos.distance_to(p); }

    /*Dessiner l'avion ???*/
    void display() const override;

    /*faire bouger l'avion*/
    void move() override;



Airport:
    /*renvoie la tour de l'aéoroport*/
    Tower& get_tower() { return tower; }

    /*dessine la tour*/
    void display() const override { texture.draw(project_2D(pos), { 2.0f, 2.0f }); }

    /*Fonction d'update*/
    void move() override
    {
        for (auto& t : terminals)
        {
            t.move();
        }
    }



Terminal:
    /*le termina l est utilisé*/
    bool in_use() const { return current_aircraft != nullptr; }

    /*Le terminal est en service (gens qui monte et qui descendent)
    bool is_servicing() const { return service_progress < SERVICE_CYCLES; }

    /* Ajouter un avion */
    void assign_craft(const Aircraft& aircraft) { current_aircraft = &aircraft; }

    /*commencer le service, vérifie les conditions*/
    void start_service(const Aircraft& aircraft)
    {
        assert(aircraft.distance_to(pos) < DISTANCE_THRESHOLD);
        std::cout << "now servicing " << aircraft.get_flight_num() << "...\n";
        service_progress = 0;
    }

    /*Cloture le service*/
    void finish_service()
    {
        if (!is_servicing())
        {
            std::cout << "done servicing " << current_aircraft->get_flight_num() << '\n';
            current_aircraft = nullptr;
        }
    }

    /**/
    void move() override
    {
        if (in_use() && is_servicing())
        {
            ++service_progress;
        }
    }


Réalisez ensuite un schéma présentant comment ces différentes classes intéragissent ensemble.
    Airport -> Terminal -> Tower -> Aircraft


Quelles classes et fonctions sont impliquées dans la génération du chemin d'un avion ?
    Waypoint, notamment la fonction getInstructinos

Quel conteneur de la librairie standard a été choisi pour représenter le chemin ?
...

Expliquez les intérêts de ce choix.
...

## C- Bidouillons !

1) Déterminez à quel endroit du code sont définies les vitesses maximales et accélération de chaque avion.

Classe AircraftTypes


Le Concorde est censé pouvoir voler plus vite que les autres avions.
Modifiez le programme pour tenir compte de cela.



2) Identifiez quelle variable contrôle le framerate de la simulation.


...

Ajoutez deux nouveaux inputs au programme permettant d'augmenter ou de diminuer cette valeur.
Essayez maintenant de mettre en pause le programme en manipulant ce framerate. Que se passe-t-il ?\
Ajoutez une nouvelle fonctionnalité au programme pour mettre le programme en pause, et qui ne passe pas par le framerate.

3) Identifiez quelle variable contrôle le temps de débarquement des avions et doublez-le.
    max_ground speed

    aircraftype.hpp 
    max_ground_speed

4) Lorsqu'un avion a décollé, il réattérit peu de temps après.
Faites en sorte qu'à la place, il soit retiré du programme.\
Indices :\
A quel endroit pouvez-vous savoir que l'avion doit être supprimé ?\   Le terminal ou la tour
Pourquoi n'est-il pas sûr de procéder au retrait de l'avion dans cette fonction ?
A quel endroit de la callstack pourriez-vous le faire à la place ?\
Que devez-vous modifier pour transmettre l'information de la première à la seconde fonction ?

5) Lorsqu'un objet de type `Displayable` est créé, il faut ajouter celui-ci manuellement dans la liste des objets à afficher.
Il faut également penser à le supprimer de cette liste avant de le détruire.
Faites en sorte que l'ajout et la suppression de `display_queue` soit "automatiquement gérée" lorsqu'un `Displayable` est créé ou détruit.
Pourquoi n'est-il pas spécialement pertinent d'en faire de même pour `DynamicObject` ?
pour les dynacmic vu que c'est un unordered set il y a moins d'interêt


6) La tour de contrôle a besoin de stocker pour tout `Aircraft` le `Terminal` qui lui est actuellement attribué, afin de pouvoir le libérer une fois que l'avion décolle.
Cette information est actuellement enregistrée dans un `std::vector<std::pair<const Aircraft*, size_t>>` (size_t représentant l'indice du terminal).
Cela fait que la recherche du terminal associé à un avion est réalisée en temps linéaire, par rapport au nombre total de terminaux.
Cela n'est pas grave tant que ce nombre est petit, mais pour préparer l'avenir, on aimerait bien remplacer le vector par un conteneur qui garantira des opérations efficaces, même s'il y a beaucoup de terminaux.\
Modifiez le code afin d'utiliser un conteneur STL plus adapté. Normalement, à la fin, la fonction `find_craft_and_terminal(const Aicraft&)` ne devrait plus être nécessaire.

## D- Théorie

1) Comment a-t-on fait pour que seule la classe `Tower` puisse réserver un terminal de l'aéroport ?

2) En regardant le contenu de la fonction `void Aircraft::turn(Point3D direction)`, pourquoi selon-vous ne sommes-nous pas passer par une réference ?


Pensez-vous qu'il soit possible d'éviter la copie du `Point3D` passé en paramètre ?

## E- Bonus

Le temps qui s'écoule dans la simulation dépend du framerate du programme.
La fonction move() n'utilise pas le vrai temps. Faites en sorte que si.
Par conséquent, lorsque vous augmentez le framerate, la simulation s'exécute plus rapidement, et si vous le diminuez, celle-ci s'exécute plus lentement.

Dans la plupart des jeux ou logiciels que vous utilisez, lorsque le framerate diminue, vous ne le ressentez quasiment pas (en tout cas, tant que celui-ci ne diminue pas trop).
Pour avoir ce type de résultat, les fonctions d'update prennent généralement en paramètre le temps qui s'est écoulé depuis la dernière frame, et l'utilise pour calculer le mouvement des entités.

Recherchez sur Internet comment obtenir le temps courant en C++ et arrangez-vous pour calculer le dt (delta time) qui s'écoule entre deux frames.
Lorsque le programme tourne bien, celui-ci devrait être quasiment égale à 1/framerate.
Cependant, si le programme se met à ramer et que la callback de glutTimerFunc est appelée en retard (oui oui, c'est possible), alors votre dt devrait être supérieur à 1/framerate.

Passez ensuite cette valeur à la fonction `move` des `DynamicObject`, et utilisez-la pour calculer les nouvelles positions de chaque avion.
Vérifiez maintenant en exécutant le programme que, lorsque augmentez le framerate du programme, vous n'augmentez pas la vitesse de la simulation.

Ajoutez ensuite deux nouveaux inputs permettant d'accélérer ou de ralentir la simulation.

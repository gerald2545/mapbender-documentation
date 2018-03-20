.. _search_router:

Search Router
*************

This element creates a configurable search formular with result output. At this time generic SQL search is supported, with more to come (WFS, Solr, ...).

.. image:: ../../../figures/search_router.png
     :scale: 80

Configuration
=============

.. image:: ../../../figures/search_router_configuration.png
     :scale: 80


The SearchRouter needs access to the database where the search tables are. You have to define a new database configuration to be able to connect with the geo database. Read more about this at `database <../../customization/database.html>`_.

**Note**: For the use of the below-described search the digitizer tables can be used. The SQL to create the tables can be found at :ref:`digitizer`.

* **Title:** Title of the element. The title will be listed in "Layouts" and allows to distinguish between different buttons. It will be indicated if "Show label" is activated.
* **Target:** Id of Map element to query.
* **Dialog:** Render inside a dialog or not.
* **Timeout factor:** Timeout factor (multiplied with autcomplete delay) to prevent autocomplete right after a search has been started.
* **Width:**  Width of the dialog (only for dialog, not sidepane)
* **Height:**  Height of the dialog (only for dialog, not sidepane)
* **Routes:** Collection of search routes.
* **Title**: Search Title (appears, when a search is added to Routes by using +)
* **Configuration**: Field to configure the search (ppears, when a search is added to Routes by using +)

You can define Searches (Routes) with the ``+`` Button. Each Search has a ``title`` which will show up in the search form in a selectbox where you can choose the search you want to use, and a ``configuration``. The definition of the search is done in YAML syntax in the textarea configuration. Here you define the database connection, the Search tables/views, the design of the form and of the result table.
The element may be integrated into the sidepane or as a button into the toolbar. To configure a button visit the documentation at `Button <../misc/button.html>`_.


Example of a route-configuration in the ``configuration`` area:

.. code-block:: yaml

    class: Mapbender\CoreBundle\Component\SQLSearchEngine
    class_options:
        connection: search_db
        relation: ortschaften
        attributes:
            - gid
            - ortsname
        geometry_attribute: geom
    form:
        ortsname:
            type: text
            options:
                required: true
            compare: exact
    results:
        view: table
        count: true
        headers:
            gid: ID
            ortsname: Name
        callback:
            event: click
            options:
                buffer: 10
                minScale: null
                maxScale: null


Comparison Mode
---------------

For every field a comparison mode can be set, which should be used by the engine when the query is send. The SQL search engine has the following modes:

* **exact:** exact comparison (key = val)
* **iexact:** comparison ignoring cases (case-insensitive)
* **like:** Standard, both-side 'like'
* **like-left:** left-hand 'like'
* **like-right:** right-hand 'like'
* **ilike**: both-side 'like', (case-insensitive - \*searchstring\*)
* **ilike-left:** left-side 'like' (case-insensitive - \*searchstring)
* **ilike-right:** right-side 'like' (case-insensitive - searchstring\*)

Styling the Results
-------------------

By default the results are shown in the default-OpenLayers Style, orange for hits and blue fo selected objects. The OpenLayer default Styling looks like this:

.. image:: ../../../figures/de/search_router_example_colour_orangeblue.png
     :scale: 80

You can overwride this by handing over a styleMap-Configuration, which could look like this:

.. code-block:: yaml

    results:
        styleMap:
            default:
                strokeColor: '#00ff00'  # Umrandungsfarbe
                strokeOpacity: 1        # 1 - opak (keine Transparenz)
                strokeWidth: 3          # Umrandingsbreite
                fillColor: '#f0f0f0'    # Füllfarbe                
                fillOpacity: 0          # Opazität Füllung, voll transparent, daher keine Füllung
                pointRadius: 6          # Größe des Punktsymbols
            select:
                strokeColor: '#0000ff'
                strokeOpacity: 1
                strokeWidth: 4
                fillColor: '#ff00ff'
                fillOpacity: 0.8
                pointRadius: 10

This definition doesnt fill the polygons because opacity is set to null (fillOpacity: 0). They are only shown with a green frame. The selected objects in this example are filled with the color green and have an opacity of 0.8, surrounded by a blue line. This color configuration looks lie this:

.. image:: ../../../figures/de/search_router_example_colour_purplegreen.png
     :scale: 80

The default settings override the OpenLayers-Default Settings, so you only have to state the things you want to overwride. If you state nothing, the default OpenLayer style will be used.

The select-style works the same way. Any statement you make woll overwrite the settings of the *final* default style.

Note, that the hexadeximal color values have to be stated in quotation marks, because the #-Symbol would be interpreted as a comment instead.

The following example creates green (unflled) objects and shows the selected object in red:

.. code-block:: yaml

    results:
        styleMap:
            default:
                strokeColor: '#00ff00'
                strokeOpacity: 1
                fillOpacity: 0
            select:
                strokeColor: '#ff0000'
                fillColor: '#ff0000'
                fillOpacity: 0.4

.. image:: ../../../figures/de/search_router_example_colour_redgreen.png
     :scale: 80

Configuration Examples
======================

In this example a search was configured for the Mapbender user and added into the sidepane, usable under the ``+`` in Layouts.

.. image:: ../../../figures/de/add_sidepane.png
     :scale: 80

The confguration dialouge for this example looks like this:

.. image:: ../../../figures/de/search_router_example_dialog.png
     :scale: 80

The element title (*Title*) is Search. It is againg diplayed as a title in the sidepane. The checkbox is unchecked, because the element is is implemented into the sidepane and not as a button. The *Timeout factor* is set to 2. The fields *Width* and *Height* are filled, but they wont be used in this application, because the element is configured in the sidepane. It is implemented via the ``+`` -Symbol and *Routes* into the search, called (*Title*) Mapbender User. The yaml-Configuration of the Element is written in *Configuration*. In Completion it reads:

.. code-block:: yaml

  class: Mapbender\CoreBundle\Component\SQLSearchEngine
  class_options:
    connection: demo        # die Datenbank, auf die das Element zugreift
    relation: mapbender_user      # die Tabelle, auf die das Element zugreift
    attributes:          # entspricht den Tabellenspalten in der Datenbank, die angesprochen werden sollen
      - gid
      - orga
      - town
      - usertype
    geometry_attribute: the_geom      # Definition der Geometrie Spalte
  form:            # ab hier beginnt die Konfiguration des Formulars
    orga:            # Feld für die Suche nach dem Namen des Mapbender Users
      type: text
      options:
        required: false        # kein Pflichtfeld
        label: 'Mapbender User'      # Überschrift über dem Feld
        attr:          # zusätzlich definierbare Attribute
          data-autocomplete: 'on'      # automatische Vervollständigung des eingetippten Suchbegriffs
          data-autocomplete-distinct: 'on'
      compare: ilike        # Vergleichsmodus
    town:            # Feld für die Suche nach der Stadt
      type: text
      options:
        required: false        # kein Pflichtfeld
        label: Stadt        # Überschrift über dem Feld
        attr:
          data-autocomplete: 'on'
          data-autocomplete-distinct: 'on'
      compare: ilike
    usertype:          # Feld für die Suche nach dem Nutzertyp
      type: choice        # Feld mit Auswahlmöglichkeiten als Dropdown
      options:
        empty_value: 'Bitte auswählen...'    # Text, der angezeigt wird, bevor etwas ausgewählt wurde
        choices:          # die Auswahlmöglichkeiten; werden wie folgt angegeben: "Eintrag in der Spalte der Datenbank": "Angezeiger Name in der Dropdown-Liste"
          1: Company
          2: Administration
          3: University
          4: User
        required: false        # kein Pflichtfeld
        label: Nutzertyp        # Überschrift über dem Feld
      compare: exact        # Vergleichsmodus
  results:          # Konfiguration der Ergebnisanzeige
    view: table          # Tabelle ausgeben
    count: true          # Anzahl der Ergebnisse anzeigen
    headers:          # Titel der Spalte; werden wie folgt angegeben: Spaltenname in der Datenbank: Bezeichnung der Spalte in der Suchausgabe der Anwendung
      gid: ID
      orga: 'Mapbender User'
      town: Stadt
    callback:
      event: click        # bei Klicken wird das Element selektiert
      options:
        buffer: 10
        minScale: null
        maxScale: 10000
    styleMap:          # Styling der Punkte in der Karte
      default:          # Styling aller angezeigten Punkte
        strokeColor: '#003366'
        strokeOpacity: 1
        fillColor: '#3366cc'
        fillOpacity: 0.5
      select:          # Styling des selektierten Objekts
        strokeColor: '#330000'
        strokeOpacity: 1
        fillColor: '#800000'
        fillOpacity: 0.5

With this configuration the search in the application looks like this:

.. image:: ../../../figures/de/search_router_example_search.png
     :scale: 80

This picture illustrates which consequences the configurations in the yaml-definition have for the search formula:

.. image:: ../../../figures/de/search_router_example_search_description.png
     :scale: 80

Displayed is the excerpt of the yaml-definition configureing the formula. Columns orga, town and usertype are used in the formula and implemented as the fields Mapebender User, Town and Usertype. Mapbender User and Town are type text, Usertype can be of various types. The text that should be displayed, if nothing is selected yet, is here "Please select…" (Nr. **1** – empty_value: ‚Please select...‘). The title above these fields is set with label (Nr. **2**). The attribute data-autocomplete: ‚on‘ results in a dropdown menu with recommendations from the database (Nr. **3**). Because compare: ilike is enabled it is not necessary to write the exact word. The search will find results that are only similar to the written term (Nr. **4** – Wheregr (the g is lowercase, nevertheless WhereGroup with uppercase G was found). The fieldtype choice is variable, possibilities are defined in choices (Nr. **5**). The table contains the possibilities as numbers (1, 2, 3, 4). In this example every number represents a text, which should be displayed in the dropdown menu.

A complete search for the Mapbender User WhereGroup, in the Town Bonn, of the Usertype Company and the found results will look like this:

.. image:: ../../../figures/de/search_router_example_search_WG.png
     :scale: 80

This picture illustrates the consequences our configuration of the yaml-defnition had on the display of the results.

.. image:: ../../../figures/de/search_router_example_results_description.png
     :scale: 80

Here only the configuration of the results is shown. The number of results is shown because count: true (Nr. **1**) is enabled. The titles of the columns are defined in headeers (Nr. **2**). Here the name of the column is mentioned first, to define which results are referenced. After the colon we set the title which will be displayed in the application. In the block styleMap the points are styled. The block default (Nr. **3**) references all points, and the block select (Nr. **4**) only selected points.

Because none of these fields are mandatory the search will work wih only on field.


Additional configuration examples
---------------------------------

Example with autocomplete and individualized display of results:

.. code-block:: yaml

   Create or Replace view brd.qry_gn250_p_ortslage as Select gid, name, gemeinde, bundesland, oba, ewz_ger,  hoehe_ger ,geom from brd.gn250_p where oba = 'AX_Ortslage' order by name;


.. code-block:: yaml

  class: Mapbender\CoreBundle\Component\SQLSearchEngine
  class_options:
      connection: search_db
      relation: brd.qry_gn250_p_ortslage
      attributes:
    - gid
    - name
    - gemeinde
    - bundesland
    - ewz_ger
    - hoehe_ger
      geometry_attribute: geom
  form:
      name:
    type: text
    options:
        required: false
        label: Name
        attr:
            data-autocomplete: on
    compare: ilike
      gemeinde:
    type: text
    options:
        required: false
    compare: ilike
  results:
      view: table
      count: true
      headers:
    name: Name
    gemeinde: Gemeinde
    bundesland: Bundesland
    ewz_ger: Einwohner
    hoehe_ger: Höhe
      callback:
    event: click
    options:
        buffer: 1000
        minScale: null
        maxScale: null
      styleMap:
    default:
        strokeColor: '#00ff00'
        strokeOpacity: 1
        fillOpacity: 0
    select:
        strokeColor: '#ff0000'
        fillColor: '#ff0000'
        fillOpacity: 0.8

Example with selection box:

.. code-block:: yaml

   Create or Replace view brd.qry_gn250_p as Select gid, name, gemeinde, bundesland, oba, geom from brd.gn250_p where oba = 'AX_Ortslage' OR oba = 'AX_Wasserlauf' order by name;


.. code-block:: yaml

  class: Mapbender\CoreBundle\Component\SQLSearchEngine
  class_options:
      connection: search_db
      relation: brd.qry_gn250_p_ortslage
      attributes:
    - gid
    - name
    - gemeinde
    - bundesland
    - oba
      geometry_attribute: geom
  form:
      oba:
    type: choice
    options:
        empty_value: 'Bitte wählen...'
        choices:
            AX_Ortslage: Ort
            AX_Wasserlauf: 'Gewässer'
      name:
    type: text
    options:
        required: false
        label: Name
        attr:
            data-autocomplete: on
    compare: ilike
      gemeinde:
    type: text
    options:
        required: false
    compare: ilike
  results:
      view: table
      count: true
      headers:
    name: Name
    gemeinde: Gemeinde
    bundesland: Bundesland
      callback:
    event: click
    options:
        buffer: 1000
        minScale: null
        maxScale: null


YAML-Definition 
---------------

In the mapbender.yml file:

.. code-block:: yaml

   target: map # ID des Kartenelements
   asDialog: true # true, Erebniswiedergabe in einem Dialogfeld
   timeoutFactor:  3 # Timeout-Faktor (multipliziert mit autcomplete Verzögerung) um die Autovervollständigung zu verhindern, nachdem eine Suche gestartet wurde
   height: 500 # Höhe des Dialogs
   width: 700 # Breite des Dialogs
   routes:    # Sammlung von Suchrouten
       demo_polygon:  # für Maschinen lesbarer Name
      class: Mapbender\CoreBundle\Component\SQLSearchEngine  #  Suchmaschine, die verwendet werden soll
      class_options:  # Diese werden an die Suchmaschine weitergegeben
          connection: digi_suche    # search_db  # DBAL Verbindungsname, der benutzt werden soll, benutzen sie ~ für default
          relation: polygons # Verbindungsauswahl, Unterabfragen können verwendet werden
          attributes: 
              - gid  # Liste von Spalten auswählen, expressions are possible
              - name 
              - type
          geometry_attribute: geom  # Name der Geometriesplate, die genutzt werden soll. Achtung: Projektion muss mit Projektion des map-Elements übereinstimmen
      form:  # Einstellungen für das Suchformular
          name:  # Feldname, Spaltenname der genutzt werden soll 
              type: text  # Eingabefeld, normalerweise Text oder Zahlen
              options:  # Einstellungen für das Eingabefeld
                  required: false  # HTML5 benötigte Attribute
                  label: Name  # benutzerdefinierte Beschriftung eingeben, sont wird die Beschriftung von dem Feldnamen abgeleitet
                  attr:  # HTML5 benötigte Attribute
                      data-autocomplete: on  # Attribut, um Autovervollständigung zu aktivieren
                      data-autocomplete-distinct: on  # Attribut, dass Autovervollständigung aktiviert aber unterscheiden lässt
                      data-autocomplete-using: type   # komma separierte Liste von anderen Eingabefeldern, in denen WHERE Angaben für die Autovervollständigung gemacht werden                
              compare: ilike  # Siehe unten für weitere Vergleichsformen
          type:
              type: choice
              options:
                  empty_value: Please select a type.
                  required: false
                  choices:
                      A: A
                      B: B
                      C: C
                      D: D
                      E: E
      results:
          view: table  # Ansicht der Ergebnisse, Ausgabe z.B. als Tabelle
          count: true # Anzahl der Treffer anzeigen
          headers:  # hBezeichnung der Tabellenüberschriften und der entsprechenden Ergebnisspalten
              gid: ID  # Spaltenname -> Überschrift
              name: Name
              type: Type
          callback:  # Was beim Klick und Mauszeiger halten passiert
              event: click  # Ergebnisliste (click oder mouseover)
              options:
                  buffer: 10    # Puffert die Geometrieergebnise (Karteneinheiten) vor dem Zoomen
                  minScale: ~   # Maßstabsbegrenzung beim Zoomen, ~ für keine Begrenzung
                        maxScale: ~
          results:
              styleMap:  # Siehe unten für weitere Styles
                  default:
                      strokeColor: '#00ff00'
                      strokeOpacity: 1
                      fillOpacity: 0
                  select:
                      strokeColor: '#ff0000'
                      fillColor: '#ff0000'
                      fillOpacity: 0.4


Class, Widget & Style
=====================

* **Class:** Mapbender\\CoreBundle\\Element\\SearchRouter
* **Widget:** mapbender.element.searchRouter.js, mapbender.element.searchRouter.Feature.js, mapbender.element.searchRouter.Search.js
* **Style:** mapbender.element.searchRouter.css


HTTP Callbacks
==============

<route_id>/autocomplete
-----------------------

Auto-completed Ajax endpoint for the predefined search route. The auto-complete uses Backbone.js. The auto-complete is implemented in mapbender.element.searchRouter.Search.js.

<route_id>/search
-----------------

Auto-completed Ajax endpoint for the predefined search route. The search module uses Backbone.js. The auto-complete is implemented in mapbender.element.searchRouter.Search.js.

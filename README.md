## Fixed map

Pour obtenir une page avec une carte fixe et du contenu que l'on peut faire défiler, comme sur [cette page](http://airbnbflo.herokuapp.com/flats), suivez les étapes ci-dessous.
Plaçons-nous (au hasard) dans le contexte d'une carte contenant des appartements à réserver.

### Step 1 - HTML

La vue correspondante doit contenir deux ```<div>```:
- la carte, donnons-lui l'id #map;
- la liste des appartements, donnons-lui la class .sidebar.

```html
# app/views/flats/index.html.erb

# la div correspondant à la carte
<div class="box-shadow" id="map"></div>

# code js pour afficher la carte et les marqueurs
<% content_for(:js) do %>
  <%= javascript_tag do %>
    $(document).on('ready', function() {
      handler = Gmaps.build('Google');
      handler.buildMap({ internal: { id: 'map' } }, function(){
        markers = handler.addMarkers(<%= raw @markers.to_json %>);
        handler.bounds.extendWith(markers);
        handler.fitMapToBounds();
      });
    })
  <% end %>
<% end %>

# la div correspondant à la liste des flats que nous voulons faire défiler
<div class="sidebar grey-bg">
  <h1 class="padding-left-15">List of flats</h1>
  <% @flats.each do |f| %>
    <div class="col-sm-12 col-md-6 index-flats">
      <h3><%= link_to f.title, flat_path(f) %></h3>
      <div class="picture-placeholder">
        <% if f.photos.first %>
          <%= image_tag f.photos.first.picture.url(:medium), class: "img-thumbnail"%>
        <% else %>
          <%= image_tag "images.png", class: "img-thumbnail flat-picture" %>
        <% end %>
      </div>
    </div>
  <% end %>
</div>
```

### Step 2 - CSS

#### La carte - #map

Afin de fixer la carte, donnons-lui les propriétés css ci-dessous, à savoir :
- une ```position: absolute;```, ```top: 50px;``` et ```bottom: 0;``` pour qu'elle soit fixée et qu'elle occupe la totalité de la hauteur de la page, bien disposée sous la navbar qui a une ```height: 50px;```,
- ```left: 0;``` ```right: 40%;``` ```width: 40%;``` pour qu'elle soit calée à gauche et qu'elle prenne 40% de la largeur du ```<body>```,
- ```z-index: 1;``` n'est pas indispensable, mais cette propriété permet de nous assurer que dans le cas où la "responsivité" du site n'est pas optimale et que des ```<div>``` seraient susceptibles de se superposer sur la carte, alors la carte resterait bien au premier plan.

#### Le contenu - .sidebar

Pour que la sidebar soit correctement disposée, donnons-lui les propriétés complémentaires à celles de #map, soit une ```width: 60%;```, et ```right: 0;```.
Ajoutons-lui maintenant la propriété ```overflow: auto``` qui permet de faire défiler son contenu.

Enfin, nous devons empêcher la "scrollbar" du body d'apparaitre, ce qui nous assure que la carte restera bien de marbre en toutes situations.

Pour cela, nous ajoutons la propriété ```overflow: hidden```.
Les autres propriétés appliquées au body ont pour but d'enlever marges et padding, pour occuper la totalité de la page.

```css
#app/assets/stylesheets/map.css.scss

html, body {
  height: 100%;
  overflow: hidden;
  margin: 0;
  padding: 0;
}

#map {
  position: absolute;
  top: 50px;
  right: 40%;
  width: 40%;
  bottom: 0;
  left: 0;
  z-index: 1;
}

.sidebar {
  position: absolute;
  width: 60%;
  top: 50px;
  right: 0;
  bottom: 0;
  overflow: auto;
  padding: 15px;
}
```


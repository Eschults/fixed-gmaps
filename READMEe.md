## Fixed map

Pour obtenir une page avec une carte fixe et du contenu que l'on peut faire défiler, comme sur [cette page](http://airbnbflo.herokuapp.com/flats), suivez les étapes ci-dessous.

### Step 1 - HTML

Le code de la vue correspondante doit contenir

```html
# app/views/flats/index.html.erb

# la div correspondant à la carte, donnons lui l'id #map.
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

# la div correspondant à la liste des flats que nous voulons faire défiler, donnons lui la classe .sidebar.
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



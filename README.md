Comment retrouver la bdd 7z gpkg du projet

se rendre ici

https://cartes.gouv.fr/rechercher-une-donnee/dataset/IGNF_BD-TOPO

dans Téléchargements et flux(152) cliquer sur

BD TOPO® V3

ensuite cliquer sur telecharger

choisir format gpkg

prendre le premier élément dispo

https://data.geopf.fr/telechargement/resource/BDTOPO?lang=FR&zone=D080&format=GPKG

https://data.geopf.fr/telechargement/resource/BDTOPO/BDTOPO_3-5_TOUSTHEMES_GPKG_LAMB93_D080_2026-06-15

permettant de telecharger

https://data.geopf.fr/telechargement/download/BDTOPO/BDTOPO_3-5_TOUSTHEMES_GPKG_LAMB93_D080_2026-06-15/BDTOPO_3-5_TOUSTHEMES_GPKG_LAMB93_D080_2026-06-15.7z


docker exec -it postgis_container \
psql -U postgres -d postgres \
-c "CREATE SCHEMA IF NOT EXISTS bdtopo;"

ogr2ogr \
  -f PostgreSQL \
  PG:"dbname=postgres host=localhost port=5432 user=postgres password=mypassword" \
  datas/BDT_3-5_GPKG_LAMB93_D080-ED2026-06-15.gpkg \
  commune \
  -nln bdtopo.commune \
  -where "code_insee_du_departement = '80'" \
  -lco GEOMETRY_NAME=geometrie \
  -nlt MULTIPOLYGON \
  -a_srs EPSG:2154 \
  --config PG_USE_COPY YES \
  -gt 200000 \
  -progress

  docker exec -it postgis_container psql \
  -U postgres \
  -d postgres \
  -c "CREATE MATERIALIZED VIEW bdtopo.commune_label_point AS SELECT fid, nom_officiel, ST_PointOnSurface(geometrie)::geometry(Point,2154) AS geometrie FROM bdtopo.commune;"

  docker exec -it postgis_container psql \
  -U postgres \
  -d postgres \
  -c "CREATE INDEX commune_label_point_geom_gist ON bdtopo.commune_label_point USING GIST (geometrie);"

  docker exec -it postgis_container psql \
  -U postgres \
  -d postgres \
  -c "SELECT f_table_schema, f_table_name, f_geometry_column, type, srid FROM geometry_columns WHERE f_table_schema = 'bdtopo' AND f_table_name = 'commune_label_point';"

  docker restart martin

  partie pour creer la view (non materialisée) des chefs lieux departementaux

  docker exec -it postgis_container psql -U postgres -d postgres -c "CREATE VIEW bdtopo.commune_chefs_lieux AS SELECT * FROM bdtopo.commune WHERE chef_lieu_de_departement = TRUE;"

  docker exec -it postgis_container psql -U postgres -d postgres -c "SELECT nom_officiel, chef_lieu_de_departement FROM bdtopo.commune_chefs_lieux;"

  docker exec -it postgis_container psql -U postgres -d postgres -c "\dv bdtopo.commune_chefs_lieux"
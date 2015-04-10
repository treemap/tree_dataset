# Tree Dataset

This PostgreSQL database contains all the trees in the United
States. It includes geospatial data about the following trees.

## Usage

    SELECT COUNT(*) FROM trees;

    SELECT geom FROM trees WHERE latin_name = 'Acer saccharum'



## Tables

  trees
    - latin_name
    - common_name
    - area
    - geom

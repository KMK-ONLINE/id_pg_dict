# id_pg_dict

Apabila besar ia pengen jadi default id dictionary bagi PostgreSQL. Sekarang ia hanya 
mengandungi daftar stop words yang mungkin berguna bagi kamu.

## Installation

1.Tentukan lokasi PG_SHARED folder kamu seperti berikut:

```
$ pg_config --sharedir
/usr/share/postgresql/9.3
```

2.Copy file indonesian.stop ke dalam folder tsearch_date

```
cp indonesian.stop /usr/share/postgresql/9.3/tsearch_data/
```

3.Dari console PostgreSQL, create search directory baru yang akan
menggunakan indonesian.stop file tadi.

```
CREATE TEXT SEARCH DICTIONARY public.simple_id_stopwords (
  TEMPLATE = pg_catalog.simple,
  STOPWORDS = indonesian
);
```

4.Ini bisa ditest seperti berikut. Kalau yang dipulangkan adalah empty array, ini bermaksud, perkataan 'yang' terkandung didalam stop word list kita.

```
postgres=# SELECT ts_lexize('public.simple_id_stopwords','yang');
 ts_lexize
 -----------
  {}
  (1 row)
```

5.Sekarang create text search configuration yang akan mengenakan dictionary yang kita create tadi.

```
CREATE TEXT SEARCH CONFIGURATION public.simple_id ( COPY = pg_catalog.simple );
ALTER TEXT SEARCH CONFIGURATION simple_id ALTER MAPPING FOR asciiword, asciihword, hword_asciipart, word, hword, hword_part WITH simple_id_stopwords;
```

6.Bandingkan hasil plainto_tsquery apabila menggunakan search configuration yang mengenakan text search configuration kita, perkataan yang terkandung di stop word list kita di strip

```
postgres=# select plainto_tsquery('pg_catalog.simple', 'sangkakala yang keluar dari langit');
plainto_tsquery
------------------------------------------------------
'sangkakala' & 'yang' & 'keluar' & 'dari' & 'langit'
(1 row)
```

```
postgres=# select plainto_tsquery('public.simple_id', 'sangkakala yang keluar dari langit');
plainto_tsquery
-------------------------
'sangkakala' & 'langit'
```


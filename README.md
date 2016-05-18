Image Plugin for Elasticsearch
==============================

The Image Plugin is an Content Based Image Retrieval Plugin for Elasticsearch using [LIRE (Lucene Image Retrieval)](https://github.com/dermotte/lire). It allows users to index images and search for similar images.

It adds an `image` field type and an `image` query

In order to install the plugin, simply run:

```sh
bin\plugin install file:<path_to>/elasticsearch-image-X.X.X-SNAPSHOT.zip
```

You can create the plugin using Maven, simply run:

```sh
JAVA_HOME=<java_home> mvn install -DskipTests=true -Dmaven.javadoc.skip=true -B -V
```

The plugin will be located at `target/releases/elasticsearch-image-X.X.X-SNAPSHOT.zip`.


|     Image Plugin          |  elasticsearch    | Release date |
|---------------------------|-------------------|:------------:|
| 2.3.2                     | 2.3.2             | 2016-05-18   |
| 2.2.0                     | 2.2.0             | 2016-04-20   |
| 1.3.0-SNAPSHOT (master)   | 1.1.0             | 2014-09-04   |
| 1.2.0                     | 1.0.1             | 2014-03-20   |
| 1.1.0                     | 1.0.1             | 2014-03-13   |
| 1.0.0                     | 1.0.1             | 2014-03-05   |


## Developers:
Kevin Wang <kzwang>

f7anty <f7anty>

OnscopeGit <OnscopeGit>

## Example
#### Create Settings

```sh
{
    "number_of_shards" : 5,
    "number_of_replicas" : 2,
    "index.version.created" : 2030299
}
```


#### Create Mapping

```sh
curl -XPUT 'localhost:9200/test/test/_mapping' -d '{
    "test": {
        "properties": {
            "my_img": {
                "type": "image",
                "feature": {
                    "CEDD": {
                        "hash": "BIT_SAMPLING"
                    },
                    "JCD": {
                        "hash": ["BIT_SAMPLING", "LSH"]
                    }
                },
                "metadata": {
                    "jpeg.image_width": {
                        "type": "string",
                        "store": "yes"
                    },
                    "jpeg.image_height": {
                        "type": "string",
                        "store": "yes"
                    }
                }
            }
        }
    }
}'
```
`type` should be `image`. This is the type register by this plugin. **Mandatory**

`feature` is a map of features for index. You can only search what you specific, e.g. base on example above, specific `JCD` with `LSH` in mapping allow search for it, but you cannot search `CEDD` with `LSH` 
because the index mapping for `LSH` is not specific and created. If you not specific hash for a `feature`, it won't work. **Mandatory, at least one is required** 

`hash` can be set if you want to search on hash. **Optional**

`metadata` is a map of metadata for index, only those metadata will be indexed. See [Metadata](#metadata). **Optional**


#### Index Image
```sh
curl -XPOST 'localhost:9200/test/test' -d '{
    "my_img": "... base64 encoded image ..."
}'
```

#### Search Image
```sh
curl -XPOST 'localhost:9200/test/test/_search' -d '{
	"from": 0,
    "size": 3,
    "query": {
        "image": {
            "my_img": {
                "feature": "CEDD",
                "image": "... base64 encoded image to search ...",
                "hash": "BIT_SAMPLING",
                "boost": 2.1,
                "limit": 100
            }
        }
    }
}'
```
`feature` should be one of the features in the mapping. See above.  **Mandatory**

`image` base64 of image to search.  **Optional if search using existing image**

`hash` should be same to the hash set in mapping. See Above.  **Optional**

`boost` score boost  **Optional**


#### Search Image using existing image in index
```sh
curl -XPOST 'localhost:9200/test/test/_search' -d '{ 	
    "query": {
        "image": {
            "my_img": {
                "feature": "CEDD",
                "index": "test",
                "type": "test",
                "id": "image1",
                "hash": "BIT_SAMPLING"
            }
        }
    }
}'
```
`index` the index to fetch image from. Default to current index.  **Optional**

`type` the type to fetch image from.  **Mandatory**

`id` the id of the document to fetch image from.  **Mandatory**

`field` the field specified as path to fetch image from. Example above is "my_img **Optional**

`routing` a custom routing value to be used when retrieving the external image doc.  **Optional**

### image query Builder
```sh
SearchRequestBuilder queryBuilder = searchClient.prepareSearch(INDEX)
		.setSearchType(SearchType.DFS_QUERY_THEN_FETCH)
		.setTypes("Image")
		.setFrom(from)
		.setSize(size);
	
	ImageQueryBuilder query = new ImageQueryBuilder("img");  //image field
	query.feature(feature);
	query.hash(hash);
	query.lookupIndex(INDEX);
	query.lookupType("Image");
	query.lookupId(itemId);	
```


### Metadata
Metadata are extracted using [metadata-extractor](https://github.com/drewnoakes/metadata-extractor/). See [SampleOutput](https://github.com/drewnoakes/metadata-extractor/wiki/SampleOutput) for some examples of metadata.

The field name in index will be `directory.tag_name`, all lower case and space becomes underscore(`_`). e.g. if the *Directory* is `JPEG` and *Tag Name* is `Image Height`, the field name will be `jpeg.image_height`



### Supported Image Formats
Images are processed by Java ImageIO, supported formats can be found [here](http://docs.oracle.com/javase/7/docs/api/javax/imageio/package-summary.html)

Additional formats can be supported by ImageIO plugins, for example [TwelveMonkeys](https://github.com/haraldk/TwelveMonkeys)


### Supported Features
[`AUTO_COLOR_CORRELOGRAM`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/AutoColorCorrelogram.java),  [`BINARY_PATTERNS_PYRAMID`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/BinaryPatternsPyramid.java), [`CEDD`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/CEDD.java), [`COLOR_LAYOUT`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/ColorLayout.java), [`EDGE_HISTOGRAM`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/EdgeHistogram.java), [`FCTH`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/FCTH.java), [`FUZZY_COLOR_HISTOGRAM`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/FuzzyColorHistogram.java), [`FUZZY_OPPONENT_HISTOGRAM`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/FuzzyOpponentHistogram.java), [`GABOR`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/Gabor.java), [`JCD`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/JCD.java), [`JOINT_HISTOGRAM`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/joint/JointHistogram.java), [`JPEG_COEFFICIENT_HISTOGRAM`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/JpegCoefficientHistogram.java), [`LOCAL_BINARY_PATTERNS`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/LocalBinaryPatterns.java), [`LOCAL_BINARY_PATTERNS_AND_OPPONENT`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/joint/LocalBinaryPatternsAndOpponent.java), [`LUMINANCE_LAYOUT`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/LuminanceLayout.java), [`OPPONENT_HISTOGRAM`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/OpponentHistogram.java), [`PHOG`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/PHOG.java), [`RANK_AND_OPPONENT`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/joint/RankAndOpponent.java), [`ROTATION_INVARIANT_LOCAL_BINARY_PATTERNS`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/RotationInvariantLocalBinaryPatterns.java), [`SCALABLE_COLOR`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/ScalableColor.java), [`SIMPLE_CENTRIST`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/centrist/SimpleCentrist.java), [`SPATIAL_PYRAMID_CENTRIST`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/centrist/SpatialPyramidCentrist.java), [`SIMPLE_COLOR_HISTOGRAM`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/SimpleColorHistogram.java), [`SPACC`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/spatialpyramid/SPACC.java), [`SPCEDD`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/spatialpyramid/SPCEDD.java), [`SPFCTH`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/spatialpyramid/SPFCTH.java), [`SPJCD`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/spatialpyramid/SPJCD.java), [`SPLBP`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/spatialpyramid/SPLBP.java), [`TAMURA`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/imageanalysis/features/global/Tamura.java)


### Supported Hash Mode
[`BIT_SAMPLING`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/indexers/hashing/BitSampling.java), [`LSH`](https://github.com/dermotte/LIRE/blob/master/src/main/java/net/semanticmetadata/lire/indexers/hashing/LocalitySensitiveHashing.java)

Hash will increase search speed with large data sets

See [Large image data sets with LIRE ?some new numbers](http://www.semanticmetadata.net/2013/03/20/large-image-data-sets-with-lire-some-new-numbers/) 


### Settings
|     Setting          |  Description    | Default |
|----------------------|-----------------|:-------:|
| index.image.use_thread_pool | use multiple thread when multiple features are required | True |
| index.image.ignore_metadata_error| ignore errors happened during extract metadata from image | True |

## ChangeLog

#### 2.3.2 (2016-05-18)

- upgrade to lire 1.0b2.
*reindex is needed if using difference version of LIRE.

#### 2.2.0 (2016-03-01)

#### 2.1.1 (2016-01-06)

#### 1.2.0 (2014-03-20)

- Use multi-thread when multiple features are required to improve index speed
- Allow index metadata
- Allow query by existing image in index

#### 1.1.0 (2014-03-13)

- Added `limit` in `image` query
- Added plugin version in es-plugin.properties

#### 1.0.0 (2014-03-05)

- initial release
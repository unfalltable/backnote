# ES

## 搜索

- 创建一个查询对象vo类，属性包括需要显示的信息，指定其属于ES中哪个索引，哪个type（表）

  - ```java
    @Document(indexName = "haoke", type = "house", createIndex = false)
    public class Data{
        //需要返回的数据
    }
    ```

- 创建一个搜索结果类，存放查询结果集合

  - ```java
    public class SearchResult{
        private Integer totalPage;
        private List<Data> list;
    }
    ```

- 创建`SearchController`

  - ```java
    public class SearchController{
        @Autowired
        private SearchService searchService;
        
        @GetMapping
        public SearchResult search(@RequestParam("keyWord") String kewWord,
                                   @RequestParam(value = "page", defaultValue = "1") Integer page){
          	//防止爬虫，保障ES性能，减少压力
            if(page > 100)  page = 1;
            return this.searchService.search(keyWord, page);
        }
    }
    ```

- 创建`SearchService`

  - ```java
    public class SearchService{
        @Autowired
        private ElasticsearchTemplate elasticsearchTemplate;
        //由后端指定页面数据条数
        public static final Integer ROWS = 10;
        
        public SearchResult serach(String kewWord, Integer page){
            //设置分页参数
            PageRequest pageRequest = PageRequest.of(page - 1, ROWS);
            SearchQuery searchQuery = new NativSearchQueryBuilder()
                .withQuery(Querybuilder.matchQuery("title", keyWord).operator(Operator.AND))
                .withPageable(pageRequest)
                .withHighlightFields(new HighlightBuilder.Field("title"))//不起作用
                .builder();
            AggregatedPage<HouseData> housePage = 
                this.elasticsearchTemplate.queryForPage(searchQuery, HouseData.class);
            return new SearchResult(housePage.getTotalPages(), housePage.getContent());
        }
    }
    ```

## 高亮显示

- 处理高亮和非高亮

```java
return new SearchResult(housePage.getTotalPages(), housePage.getContent(), new SearchResultMapper(){
@Override
public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> clazz, Pageable pageable) throws Exception{
    //如果查询到的条数为0时返回空对象
    if(response.getHits().totalHits == 0) 
        return new AggregatedPageImpl<>(Collection.emptyList(), pageable, 0L);
    //接收
    List<T> list = new ArrayList<>();
    //遍历命中的数据
    for(SearchHits searchHit : response.getHits().getHits()){
        T obj = (T) ReflectUtils.newInstance(clazz);
        FieldUtils.writeField(obj, "id", searchHit.getId(), true);

        //非高亮字段的数据处理
        for(Map.Entry<String, Object> entry : searchHit.getSourceAsMap().entrySet()){
            if(null == FieldUtils.getFiled(clazz, entry.getKey(), true)) continue;
            FieldUtils.writeField(obj, entry.getKey(), entry.getValue(), true);
        }
        //处理高亮字段
        for(Map.Entry<String, HighlighField> entry : searchHit.getHighlighFields().entrySet()){
            StringBulider sb = new StringBuilder();
            Text[] fragments = entry.getValue().getFragments();
            for(Text fragment : fragments){
                sb.append(fragment.toString());
            }
            FieldUtils.writeField(obj, entry.getKey(), sb.toString(), true);
        }
        list.add(obj);
    }
    return new AggregatedPageImpl<>(list, pagable, response.getHits().totalHits);
});                                                           
```

## 热词搜索

- redis使用sortedSet

- `SearchController` 添加方法，并添加热词搜索次数

  - ```java
    public class SearchController{
        @Autowired
        private SearchService searchService;
        @Autowired
        private RedisTemplate redisTemplate;
        
        @GetMapping
        public SearchResult search(){
            if(page > 100)  page = 1;
            SearchResult search = this.searchService.search(keyWord, page);
            
            //热词加入redis
            String redisKey = "SEARCH_HOT_WORDS";
            //数据条数 = 数据页数 * 每页条数 + 当前页数据条数
            int count = ((Math.max(search.getTotalPage(), 1) - 1) * searchService.ROWS) + search.getList().size();
            this.redisTemplate.opsForZSet().add(redisKey, keyWord, count);
            //判断是否返回热词搜索
            if(search.totalPage() <=1)[
                Set set = this.redisTemplate.opsForZSet().reverseRange(redsikey, 0, 4);
                search.setHotWord(set);
            ]
            
            return search;
        }
    }
    ```

- 修改`SearchResult`

  - ```java
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class SearchResult{
        private Integer totalPage;
        private List<HouseData> list;
        //新增一个热词字段
        private Set<String> hotWord;
    }
    ```

## 拼音分词

- 创建索引时使用拼音分词插件，配置拼音插件

- 创建索引时为字段建立一个拼音的子字段，该子字段使用拼音分词器

  - ```json
    "properties": {
        "title": {
            "type": "text",
            "analyzer": "ik_max_word",
            "fields": {
                "type": "text",
            	"analyzer": "pinyin_analyzer"
            }
        },
        .............
    }
    ```

- 高亮会失效，使用混合搜索可解决

  - ```json
    "query":{
    	"mult_match": {
            "query": "地铁kou",
            "fields": [
                "title",
                "title.pinyin"
            ]
        },
    	"highlight": {
            "fields": {
                "title.pinyin":{},
                'title':{}
            }
        }
    }
    ```

- 修改 `SearchService` 中的实现

  - ```java
    public class SearchService{
        @Autowired
        private ElasticsearchTemplate elasticsearchTemplate;
        //由后端指定页面数据条数
        public static final Integer ROWS = 10;
        
        public SearchResult serach(){
            //设置分页参数
            PageRequest pageRequest = PageRequest.of(page - 1, ROWS);
            SearchQuery searchQuery = new NativSearchQueryBuilder()
                //这里修改为混合搜索
                .withQuery(Querybuilder.multMatchQuery(keyWord, "title", "title.pinyin").operator(Operator.AND))
                .withPageable(pageRequest)
                .withHighlightFields(new HighlightBuilder.Field("title"))
                .builder();
            AggregatedPage<HouseData> housePage = 
                this.elasticsearchTemplate.queryForPage(searchQuery, HouseData.class);
            return new SearchResult(housePage.getTotalPages(), housePage.getContent());
    }
    ```

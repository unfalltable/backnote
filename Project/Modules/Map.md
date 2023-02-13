# Mongo实现

## 地图显示附近房源位置

- 需要返回给前端 房源集合（需要有经纬度信息），可以给房源添加经纬度信息，或者创建一个新的类保存（位置集合（X，Y），id，房源id，房源标题）

- 在MongoDB中为经纬度信息创建地理位置索引

  - `db.house.createIndex({loc:"id"})`

- 需要存储地图缩放比对应的公里数

- 查询MongoDB

  - ```java
    @Service
    public class MongoHouseService{
        //需要定义缩放比对应的公里数
        public static final Map<Integer, Double> BAIDU_ZOOM = new HashMap<>();
        @Autowired
        private MongoTemplate mongoTemplate;
        
        public MapHouseDataResult queryHouseData(Float lng, Float lat, Integer zoom){
            //1.5倍距离范围
            double distance = BAIDU_ZOOM.get(zoom) * 1.5 / 111.12;
            
            Query query = Query.query(Criteria.where("loc")
                                      .near(new Point(lng, lat))
                                      .maxDistance(distance));
            List<MongoHouse> mongoHouses = this.mongoTemplate.find(query, MongoHouse.class);
            List<MapHouseXY> list = new ArrayList<>();
            for(MongoHouse mongoHouse : mongoHouses){
                list.add(new MapHouseXY(mongoHouses.getLoc()[0], mongoHouse.getLoc()[1]));
            }
            return new MapHouseDataResult(list);
        }
    }
    ```

# Redis实现 

## 附近商铺

### Redis中的GEO实现

# 百度地图API

## 简介

- 实现地图找房
- 创建地图、覆盖物、控件、事件
- 坐标转换、路线规划、IP定位

## 地图找房

- 

## 运动类

- 运动轨迹、保存路线、分享路线、附近的人、附近路线
- 基于百度地图的鹰眼轨迹服务实现

## 物流

- 运行轨迹、电子wei'l
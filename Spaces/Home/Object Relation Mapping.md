# Spring Data JPA

> Spring에서 제공하는 ORM 구현체

## 개괄

> DB의 Relation을 프로그래밍 언어의 객체처럼 다루기 위해 매핑하는 기술
> 개발자는 DB에 매핑된 Entity를 컨트롤하여 DB의 Record를 다룰 수 있다.


### Common

> build.gradle
```
	implementation 'org.hibernate.orm:hibernate-spatial:6.4.1.Final'
	implementation 'org.geotools:gt-shapefile:29.0'
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

> properties
```
spring.datasource.url=jdbc:sqlserver://192.168.0.150;databaseName=DB_IAS;loginTimeout=0;integratedSecurity=false;SelectMethod=Cursor;encrypt=true;trustServerCertificate=true
spring.datasource.username= sample_user
spring.datasource.password= sample321!

#JPA
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.SQLServerDialect
spring.jpa.hibernate.naming.physical-strategy = org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

### Repository
```java
public interface CatalogRepository extends JpaRepository<CatalogEntity, String>, JpaSpecificationExecutor<CatalogEntity>{}
```

### Entity
```java
@Entity
@Table(name="TB_Catalog")
@AllArgsConstructor
@NoArgsConstructor
@Data
public class CatalogEntity {
    @Id
    @Column(name = "CatalogID")
    private String catalogId;

    @Column(name = "ImagingStartTime")
    private LocalDateTime imagingStartTime;

    @Column(name = "ImagingCenterTime")
    private LocalDateTime imagingCenterTime;

    @Column(name = "ImagingEndTime")
    private LocalDateTime imagingEndTime;

    @Column(name = "ProductLevel")
    private String productLevel;

    @Column(name = "ProductFormat")
    private String productFormat;

    @Column(name = "ProductType")
    private String productType;

    @Column(name = "Orbit")
    private Integer orbit;

    @Column(name = "Satellite")
    private String satellite;

    @Column(name = "Sensor")
    private String sensor;

    @Column(name = "Geometry", columnDefinition = "geometry")
    private Geometry geometry;

    @Column(name = "BrowsePath")
    private String browsePath;

    @Column(name = "ThumbPath")
    private String thumbPath;

    @Column(name = "CloudCoverAvg")
    private Integer cloudCoverAvg;

    @Column(name = "IsDel")
    private char isDel;

    @Column(name = "RegUser")
    private String regUser;

    @Column(name = "RegDtime")
    private LocalDateTime regDtime;

    @Column(name = "ModifyUser")
    private String modifyUser;

    @Column(name = "ModifyDtime")
    private LocalDateTime modifyDtime;
}
```
### Specification
```java
public class CatalogSpecification {
	
	// between imagingCenterTime #{startTime} and #{endTime}
    public static Specification<CatalogEntity> imagingCenterTimeBetween(LocalDateTime startTime, LocalDateTime endTime) {
        return (root, query, cb) -> cb.between(root.get("imagingCenterTime"), startTime, endTime);
    }
    
	// between imagingCenterTime #{startTime} and #{endTime}
    public static Specification<CatalogEntity> regTimeBetween(LocalDateTime startTime, LocalDateTime endTime) {
        return (root, query, cb) -> {
        	if(startTime == null || endTime == null) {
        		return cb.isTrue(cb.literal(true));
        	}
        	return cb.between(root.get("regDtime"), startTime, endTime);
        };
    }

    // productLevel in (#{item}, #{item}, ...}
    public static Specification<CatalogEntity> productLevelIn(List<String> productLevels) {
        return (root, query, cb) -> {
            if (productLevels == null || productLevels.isEmpty()) {
                return cb.isTrue(cb.literal(true)); // 항상 참인 조건
            }
            return root.get("productLevel").in(productLevels);
        };
    }
    
    // productLevel = #{item}
    public static Specification<CatalogEntity> productLevelIs(String productLevel) {
        return (root, query, cb) -> {
            if (productLevel == null || productLevel.isEmpty()) {
                return cb.isTrue(cb.literal(true)); // 항상 참인 조건
            }
            return cb.equal(root.get("productLevel"), productLevel);
        };
    }
    
    // productLevel in (#{item}, #{item}, ...}
    public static Specification<CatalogEntity> satelliteIn(List<String> satellites) {
        return (root, query, cb) -> {
        	if(satellites == null || satellites.isEmpty()) {
        		return cb.isTrue(cb.literal(true));
        	}
        	return root.get("satellite").in(satellites);
        };
    }
    
    // satellite = #{item}
    public static Specification<CatalogEntity> satelliteIs(String satellite) {
        return (root, query, cb) -> {
        	if(satellite == null || satellite.isEmpty()) {
        		return cb.isTrue(cb.literal(true));
        	}
        	return cb.equal(root.get("satellite"), satellite);
        };
    }
    
    // geometry.STIntersects( geometry::STGeomFromText( #{wtkGeom}, '4326' ) ) = 'true'
    public static Specification<CatalogEntity> intersects(String wktGeom) {
        return (root, query, cb) -> {
            if (wktGeom == null || wktGeom.isEmpty()) {
                return cb.isTrue(cb.literal(true)); // 무조건 참인 조건
            }
            
            try {
            	new WKTReader2().read(wktGeom);
            } catch (ParseException e) {
                throw new IllegalArgumentException("Invalid WKT", e);
            }
            
            
            
            // WKT 문자열을 Geometry 타입으로 변환
            Expression<String> geomFromText = cb.function(
                "geometry::STGeomFromText", 
                String.class, 
                cb.literal(wktGeom),
                cb.literal("4326") // 좌표계 참조 시스템 ID
            );

            // STIntersects 함수 사용
            Expression<Boolean> stIntersects = cb.function(
                "ST_Intersects",
                Boolean.class,
                root.get("geometry"),
                geomFromText
            );

            return cb.equal(stIntersects, true) ; // 부울 결과로 비교            
        };
    }
    
	// between cloudCoverAvg #{start} and #{end}
    public static Specification<CatalogEntity> cloudCoverAvgBetween(Integer start, Integer end) {
        return (root, query, cb) -> {
        	if(start == null || end == null) {
        		return cb.isTrue(cb.literal(true));
        	}
        	return cb.between(root.get("cloudCoverAvg"), start, end);
        };
    }
    
	// isDel = #{item}
    public static Specification<CatalogEntity> isDelIs(char isDel) {
        return (root, query, cb) -> 
        	 cb.equal(root.get("isDel"), isDel);
    }
}

```

### Service
```java
    public List<CatalogEntity> findByParams(CatalogRequest req) {
        Specification<CatalogEntity> spec = Specification
            .where(CatalogSpecification.imagingCenterTimeBetween(req.getStartTime(), req.getEndTime()))
            .and(CatalogSpecification.productLevelIn(req.getProductLevel()))
            .and(CatalogSpecification.satelliteIs(req.getSatellite()))
            .and(CatalogSpecification.intersects(req.getGeom().toString()))
            .and(CatalogSpecification.cloudCoverAvgBetween(req.getCloudCoverageMin(), req.getCloudCoverageMax()))
            .and(CatalogSpecification.isDelIs(req.getIsDel()))
            ;

        return catalogRepository.findAll(spec);
    }	
    
    public Page<CatalogEntity> findByParams(CatalogRequest req, Pageable pageable) {
        Specification<CatalogEntity> spec = Specification
            .where(CatalogSpecification.imagingCenterTimeBetween(req.getStartTime(), req.getEndTime()))
            .and(CatalogSpecification.productLevelIn(req.getProductLevel()))
            .and(CatalogSpecification.satelliteIs(req.getSatellite()))
            .and(CatalogSpecification.intersects(req.getGeom().toString()))
            .and(CatalogSpecification.cloudCoverAvgBetween(req.getCloudCoverageMin(), req.getCloudCoverageMax()))
            .and(CatalogSpecification.isDelIs(req.getIsDel()))
            ;

        return catalogRepository.findAll(spec, pageable);
    }	
```

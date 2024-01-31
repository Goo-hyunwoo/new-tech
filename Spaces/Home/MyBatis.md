# MyBatis Framework

> JAVA와 DB의 연동 시, 데이터 매핑과 요청을 관리해주는 라이브러리
> XML(Query, Config), Interface(Mapper)

> SqlSessionFactoryBuilder: Factory를 생성하기 위한 Singleton Builder
> SqlSessionFactory : Singleton으로 관리되는 Session 생성 객체
> SqlSession: Thread - Transaction Mapping

> SqlSession에 Mapper를 연결하고, Mapper 인터페이스에 선언되어 매핑된 쿼리를 실행한다.

### 다중 데이터베이스 연결
```java
@Configuration
@MapperScan(basePackages = {"pixo.edu.com.domain.*.mapper"})
@EnableTransactionManagement
public class DababaseConfig {
		
	    @Bean
	    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
	        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
	        sessionFactory.setDataSource(dataSource);
	        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
	        sessionFactory.setMapperLocations(resolver.getResources("pixo/edu/com/domain/*/mapper/*.xml"));
	        return sessionFactory.getObject();
	    }

	    @Bean
	    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) throws Exception {
	        final SqlSessionTemplate sqlSessionTemplate = new SqlSessionTemplate(sqlSessionFactory);
	        return sqlSessionTemplate;
	    }
	    
		/*
		@Bean
		public DataSource dataSource() {
			return DataSourceBuilder
						.create()
						.url(null)
						.driverClassName(null)
						.username(null)
						.password(null)
						.build();
		}
		*/
}
```

> 다중 데이터베이스 연결에서 SqlSessionFactory 마다 한 개의 Datasource를 매핑해야 한다.
> 즉 Factory 패턴에서 Session을 만들 때, Datasource를 교환해가며 Factory를 유지할 수는 없다.


### Handler Mapping
> 일부 자료 구조는 MyBatis에서 기본적으로 바인딩 되지 않는다.
> List 등의 DB 자료구조와 java의 List를 매핑하기 위해서 아래와 같은 TypeHandler를 생성한다.

> StringListTypeHandler.java
```java
@MappedTypes(java.util.ArrayList.class)
@MappedJdbcTypes(JdbcType.ARRAY)
public class StringListTypeHandler extends BaseTypeHandler<List<String>> {
	@Override
	public void setNonNullParameter(PreparedStatement ps, int i, List<String> parameter, JdbcType jdbcType)
			throws SQLException {
		if(parameter == null || parameter.size() == 0) {
			ps.setArray(i,  null);
			return;
		}
		String[] datum = new String[parameter.size()];
		for(int num = 0; num < parameter.size(); num++) {
			datum[num] = parameter.get(num);
		}
		ps.setArray(i, ps.getConnection().createArrayOf("varchar", datum));
		
	}

	@Override
	public List<String> getNullableResult(ResultSet rs, String columnName) throws SQLException {
		return getArrayListFromSqlArray(rs.getArray(columnName));
	}

	@Override
	public List<String> getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
		return getArrayListFromSqlArray(rs.getArray(columnIndex));
	}

	@Override
	public List<String> getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
		return getArrayListFromSqlArray(cs.getArray(columnIndex));
	}
	
	private List<String> getArrayListFromSqlArray(Array array) throws SQLException {
		if(array == null) {
			return null;
		}
		String[] datum = (String[]) array.getArray();
		if(datum == null) {
			return null;
		}
		List<String> list = new ArrayList<>();
		for (String d : datum) {
			list.add(d);
		}
		return list;
	}
```

> DoubleListTypeHandler.java
```java
@MappedTypes(java.util.ArrayList.class)
@MappedJdbcTypes(JdbcType.ARRAY)
public class DoubleListTypeHandler extends BaseTypeHandler<List<Double>> {
	@Override
	public void setNonNullParameter(PreparedStatement ps, int i, List<Double> parameter, JdbcType jdbcType)
			throws SQLException {
		if (parameter == null || parameter.size() == 0) {
			ps.setArray(i, null);
			return;
		}
		Double[] datum = new Double[parameter.size()];
		for (int num = 0; num < parameter.size(); num++) {
			datum[num] = parameter.get(num);
		}
		ps.setArray(i, ps.getConnection().createArrayOf("double", datum));

	}

	@Override
	public List<Double> getNullableResult(ResultSet rs, String columnName) throws SQLException {
		return getArrayListFromSqlArray(rs.getArray(columnName));
	}

	@Override
	public List<Double> getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
		return getArrayListFromSqlArray(rs.getArray(columnIndex));
	}

	@Override
	public List<Double> getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
		return getArrayListFromSqlArray(cs.getArray(columnIndex));
	}

	private List<Double> getArrayListFromSqlArray(Array array) {
		List<Double> list = null;
		if (array == null) {
			return list;
		}
		try {
			try {
				Double[] temp = (Double[]) array.getArray();
				if (temp == null) {
					return list;
				}
				list = Arrays.stream(temp).collect(Collectors.toList());
			} catch (Exception e) {
				String[] temp = (String[]) array.getArray();
				if (temp == null) {
					return list;
				}
				list = Arrays.stream(temp).map(Double::parseDouble).collect(Collectors.toList());
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
		return list;
	}
}
```

## POOLED / UNPOOLED

> UNPOOLED: 매 요청에 대해 커넥션을 열고 닫는 간단한 Datasource
> POOLED: 풀링이 적용된 JDBC Connection으로, 빠른 응답을 요구할 때 사용


## Dynamic Query

> 다이나믹 쿼리 구문은 정해진 형식을 이용하여 사용
> if / where / foreach / ....
# Kafka-spark
Code samples of kafka, spark and mongodb


import java.sql.Timestamp;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.GregorianCalendar;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.Time;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;

import scala.Tuple2;

import com.infy.gs.automation.beans.AlertBean;
import com.infy.gs.automation.dao.AlertDAO;



public class AlertConsumerService {
	public static final int MINUTE_ROUND_INTERVAL = 30;
	
	
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		long durationInMilliSec = 30000;
        Map<String,Integer> topicMap = new HashMap<String,Integer>();
        String[] topic = "test,".split(",");
        for(String t: topic)
        {
            topicMap.put("test", new Integer(1));
        }
      
        
        SparkConf _sparkConf = new SparkConf().setAppName("KafkaQueueReceiver").setMaster("local[2]").set("spark.driver.host","zeus07").set("spark.driver.port","8080"); 
       _sparkConf.set("spark.cassandra.connection.host", "localhost");
        
       JavaSparkContext sc = new JavaSparkContext(_sparkConf); 
        
        JavaStreamingContext jsc = new JavaStreamingContext(sc,
				new Duration(durationInMilliSec));
        
					        
        
        JavaPairReceiverInputDStream<String, String> messages = (JavaPairReceiverInputDStream<String, String>) KafkaUtils.createStream(jsc, "localhost:2181","test-consumer-group", topicMap );

		
		
		   JavaDStream<String> data = messages.map(new Function<Tuple2<String, String>, String>() 
		            {
		                public String call(Tuple2<String, String> message)
		                {
		                	
		                  System.out.println("whole message ======================="+message._2());
		                   return message._2();
		                }
		            }
		      );
		
		   

		data.foreachRDD(new Function2<JavaRDD<String>, Time, Void>() {
		      @Override
		      public Void call(JavaRDD<String> rdd, Time time) {
		        
		    	  AlertDAO dao = new AlertDAO();
		        
		        JavaRDD<AlertBean> rowRDD = rdd.map(new Function<String, AlertBean>() {
		          public AlertBean call(String word) throws ParseException {
		        	  
		        	  String[] xr = word.split(",");
		        	  
		        	 
		        	  
		        	  String alertTypevalue = CommaFmData(xr[0]);
		        	  String processnameval = CommaFmData(xr[1]);
		        	  String categoryval = CommaFmData(xr[2]);
		        	  String jobnameval = CommaFmData(xr[3]);
		        	  String eventtimeval = CommaFmData(xr[4]);
		        	  String tierval = CommaFmData(xr[5]);
		        	  String applicationFamilyval = CommaFmData(xr[6]);
		        	  String nodeval = CommaFmData(xr[7]);
		        	  String subBusinessUnitval = CommaFmData(xr[8]);
		        	  String nativeBusinessUnitval = CommaFmData(xr[9]);
		        	  String alertDetailval = CommaFmData(xr[10]);
		        	  String applicationNameval = CommaFmData(xr[11]);
		        	        	 
		        	  
		        	  Timestamp strTimestamp=strTimestamp(eventtimeval);
		        	    
		        	  AlertBean record = new AlertBean(subBusinessUnitval, categoryval,alertTypevalue, alertDetailval, applicationNameval, applicationFamilyval, tierval, nodeval, nativeBusinessUnitval,jobnameval,processnameval,strTimestamp);
		      
		              return record;
		          }
		          
		          
		        });
		        List<AlertBean> list =  rowRDD.collect();
		
			      
			      for(AlertBean bean :list){
			    	  dao.createAlert(bean);
			      }
		     
		        return null;
		      }
		    });

	    jsc.start();
	    jsc.awaitTermination();
	}
    	 
	private static String CommaFmData(String msg) {
	       String value="";
	      
		
			  if(msg.contains("=")&& msg!=null){
	        	   String alertTypekey[] = msg.split("=");
			       value = alertTypekey[1];
       	  }
			  
		
       return value;
   }
	
 public static Timestamp strTimestamp(String strtime) throws ParseException {
	 System.out.println("strtime8888888888888888888888888888"+strtime);
	
	 		SimpleDateFormat sdf= new SimpleDateFormat("MM/dd/yyyy HH:mm");
	 		
	        java.util.Date d = sdf.parse(strtime);
	        d = truncateToLeft(d);
	        Timestamp  ts = new Timestamp(d.getTime());
			return ts;
	    }
 
private static java.util.Date truncateToLeft(java.util.Date date){            
         Calendar c = new GregorianCalendar();
       c.setTime(date);        
       int interval = 60 / MINUTE_ROUND_INTERVAL;            
         for(int i=0; i<interval; i++){
               if (c.get(Calendar.MINUTE) > i*MINUTE_ROUND_INTERVAL && c.get(Calendar.MINUTE) < (i+1)*MINUTE_ROUND_INTERVAL){
                 c.set(Calendar.MINUTE, i*MINUTE_ROUND_INTERVAL);
               }
       }       
       c.set(Calendar.SECOND, 0);
       c.set(Calendar.MILLISECOND, 0);
       date = c.getTime();
       return date;
   }
   

   private static java.util.Date truncateToRight(java.util.Date date){           
         Calendar c = new GregorianCalendar();
       c.setTime(date);        
       int interval = 60 / MINUTE_ROUND_INTERVAL;
            
         for(int i=0; i<interval; i++){
               if (c.get(Calendar.MINUTE) > i*MINUTE_ROUND_INTERVAL && c.get(Calendar.MINUTE) < (i+1)*MINUTE_ROUND_INTERVAL){
                 c.set(Calendar.MINUTE, (i+1)*MINUTE_ROUND_INTERVAL);
               }
       }
         
       c.set(Calendar.SECOND, 0);
       c.set(Calendar.MILLISECOND, 0);
       date = c.getTime();
       return date;
   }


}

Feature Extraction Service



import java.util.ArrayList;
import java.util.List;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.PairFunction;

import scala.Tuple2;

import com.google.gson.Gson;
import com.infy.gs.automation.beans.AlertBean;
import com.infy.gs.automation.beans.ErrorBean;
import com.infy.gs.automation.beans.FeatureExtractionBean;
import com.infy.gs.automation.beans.JoinKeyBean;
import com.infy.gs.automation.beans.ProcessBean;
import com.infy.gs.automation.dao.AlertDAO;
import com.infy.gs.automation.dao.ErrorDAO;
import com.infy.gs.automation.dao.FeatureExtractionDAO;
import com.infy.gs.automation.dao.ProcessDAO;

public class FeatureExtractionService {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
	
		AlertDAO alertDao = new AlertDAO();
		ErrorDAO errorDAO = new ErrorDAO();
		ProcessDAO processDAO = new ProcessDAO();
		FeatureExtractionDAO featureExtractionDAO = new FeatureExtractionDAO();
		Gson gson = new Gson();
		AlertBean bean= new AlertBean();
		
		List<AlertBean> alertList = alertDao.retriveAlert(new AlertBean());
		
		List<ErrorBean> errorList = errorDAO.retriveError(new ErrorBean());
		
		
		System.out.println("Size of lists.........................................."+alertList.size()+"::::::"+errorList.size());
		
		
		  SparkConf _sparkConf = new SparkConf().setAppName("KafkaQueueReceiver").setMaster("local[2]").set("spark.driver.host","zeus07").set("spark.driver.port","8080"); 
	       _sparkConf.set("spark.cassandra.connection.host", "localhost");
	        
	       JavaSparkContext sc = new JavaSparkContext(_sparkConf); 
	        
	      
	       
	      
	       List<Tuple2<String, ErrorBean>> keyErrorList = new ArrayList<Tuple2<String, ErrorBean>>();
	       
	       
	       for(ErrorBean error  :errorList){
	    	   ProcessBean criteria = new ProcessBean();
	    	   criteria.setLogFileName(error.getLogFileName());
			   List<ProcessBean> values = processDAO.retriveProcess(criteria);	
			   values.get(0).getProcessName();
			   JoinKeyBean key = new JoinKeyBean(error.getEventtime(), values.get(0).getProcessName());	
			   keyErrorList.add(new Tuple2<String, ErrorBean>(gson.toJson(key), error ));
	       }
	       
	       
	       JavaRDD<Tuple2<String, ErrorBean>> rdderror =  sc.parallelize(keyErrorList);
	       
	       System.out.println("Size of lists.......................keyErrorList..................."+keyErrorList.size());
	       
	       List<Tuple2<String, AlertBean>> keyAlertList = new ArrayList<Tuple2<String, AlertBean>>();
	       
	       
	       for(AlertBean alert  :alertList){	
	    	   JoinKeyBean key = new JoinKeyBean(alert.getEventtime(), alert.getProcessname());
			   keyAlertList.add(new Tuple2<String, AlertBean>(gson.toJson(key), alert ));
	       }
	       
	       JavaRDD<Tuple2<String, AlertBean>> rddalert= sc.parallelize(keyAlertList);
	       
	       JavaPairRDD<String, AlertBean>alertPair = rddalert.mapToPair(new PairFunction<Tuple2<String, AlertBean>, String, AlertBean>() {	             
					@Override
					public Tuple2<String, AlertBean> call(Tuple2<String, AlertBean> keyerror) throws Exception {		
						
						return new Tuple2<String, AlertBean>(keyerror._1, keyerror._2 ); 
					}
		           });
	       
	       
	       JavaPairRDD<String, ErrorBean> errorPair = rdderror.mapToPair(new PairFunction<Tuple2<String, ErrorBean>, String, ErrorBean>() {
	             
					@Override
					public Tuple2<String, ErrorBean> call(Tuple2<String, ErrorBean> keyerror) throws Exception {		
						
						return new Tuple2<String, ErrorBean>(keyerror._1, keyerror._2 ); 
					}
		           });
	       
		
	       JavaPairRDD<String, Tuple2<AlertBean,ErrorBean>> joinPair =  alertPair.join(errorPair);
	       
	       System.out.println("Size of lists.......................JoinPair..................."+joinPair.collect().size());
	       
	       
	       List<Tuple2<String, Tuple2<AlertBean,ErrorBean>>> listJoins = joinPair.collect();
	       for(Tuple2<String, Tuple2<AlertBean,ErrorBean>> tuple :listJoins){
	    	   JoinKeyBean joinbean = gson.fromJson(tuple._1,JoinKeyBean.class);
	    	   AlertBean albean = tuple._2._1;
	    	   ErrorBean erbean = tuple._2._2;
	    	   FeatureExtractionBean febean = new FeatureExtractionBean(						
	    			   joinbean.getProcessName(),joinbean.getEventtime(), erbean.getLogmessage(),
	    			   albean.getAlertDetail(), albean.getCategory()
	    			   				);
	    	   featureExtractionDAO.createExtraction(febean);;
	       }
	       
       
	       /*
	       
	       publipublic FeatureExtractionBean(String processName, Timestamp eventTime,
	   			String error, String alertDetail, String alertCategory) {*/
	        
	}
	
	
}



import java.sql.Timestamp;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.Time;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;

import com.google.gson.Gson;
import com.infy.gs.automation.beans.ErrorBean;
import com.infy.gs.automation.beans.LogData;
import com.infy.gs.automation.dao.ErrorDAO;

import scala.Tuple2;



public class LogConsumerService {

	
	
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		long durationInMilliSec = 30000;
        Map<String,Integer> topicMap = new HashMap<String,Integer>();
        String[] topic = "test,".split(",");
        for(String t: topic)
        {
            topicMap.put("test", new Integer(1));
        }
      
        
        SparkConf _sparkConf = new SparkConf().setAppName("KafkaQueueReceiver").setMaster("local[2]").set("spark.driver.host","zeus07").set("spark.driver.port","8080"); 
       _sparkConf.set("spark.cassandra.connection.host", "localhost");
        
       JavaSparkContext sc = new JavaSparkContext(_sparkConf); 
        
        JavaStreamingContext jsc = new JavaStreamingContext(sc,
				new Duration(durationInMilliSec));
        
					        
        
        JavaPairReceiverInputDStream<String, String> messages = (JavaPairReceiverInputDStream<String, String>) KafkaUtils.createStream(jsc, "localhost:2181","test-consumer-group", topicMap );

		
		
		   JavaDStream<LogData> data = messages.map(new Function<Tuple2<String, String>, LogData>() 
		            {
		                public LogData call(Tuple2<String, String> message)
		                {
		                	System.out.println("whole string**********"+message._2());
		                   String json = message._2();
		                   Gson gson = new Gson();
		                   LogData data = gson.fromJson(json, LogData.class);
		                   System.out.println(data.getMessage()+"Message------------------------");
		                   return data;
		                }
		            }
		      );
		
	

	data.foreachRDD(new Function2<JavaRDD<LogData>, Time, Void>() {
		      @Override
		      public Void call(JavaRDD<LogData> rdd, Time time) {
		        
		    	  ErrorDAO dao = new ErrorDAO();
		        
		        JavaRDD<ErrorBean> rowRDD = rdd.map(new Function<LogData, ErrorBean>() {
		          public ErrorBean call(LogData data) {
		        	  
		        	 
		        	  
		        	  ErrorBean record = new ErrorBean(data.getTimestamp(), data.getLogfileName(),data.getLevelPattren(), data.getLoggerName(), data.getThreadName(), data.getMessage());
		      
		              return record;
		          }
		          
		          
		        });
		        List<ErrorBean> list =  rowRDD.collect();
		
			      
			      for(ErrorBean bean :list){
			    	  dao.createError(bean);
			      }
		     
		        return null;
		      }
		    });

	    jsc.start();
	    jsc.awaitTermination();
	}
    	 
	private static String CommaFmData(String msg) {
	       String value="";
	      
		
			  if(msg.contains("=")&& msg!=null){
	        	   String alertTypekey[] = msg.split("=");
			       value = alertTypekey[1];
       	  }
			  
		
       return value;
   }
	
	 public static Timestamp strTimestamp(String strtime) {
	    //    String text = "2015-06-11 14:55:05.123456";
	        Timestamp ts = Timestamp.valueOf(strtime);
	        System.out.println(ts.getNanos());
			return ts;
	    }
	
}



import java.net.UnknownHostException;
import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;


import com.infy.gs.automation.beans.AlertBean;
import com.infy.gs.automation.util.MongoConnection;
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;
import com.mongodb.util.JSON;

public class AlertDAO {
	
	
	DB db = MongoConnection.getConnection();
	DBCollection dbCollection = db.getCollection("alert");
	
	public void createAlert(AlertBean record) {		
		BasicDBObject basicDBObject = new BasicDBObject();
		basicDBObject.put("subBusinessUnit",
				record.getSubBusinessUnit());
		basicDBObject.put("category", record.getCategory());
		basicDBObject.put("alertType", record.getAlertType());
		basicDBObject.put("alertDetail", record.getAlertDetail());
		basicDBObject.put("applicationName",
				record.getApplicationName());
		basicDBObject.put("applicationFamily",
				record.getApplicationFamily());
		basicDBObject.put("tier", record.getTier());
		basicDBObject.put("node", record.getNode());
		basicDBObject.put("nativeBusinessUnit",
				record.getNativeBusinessUnit());
		
		basicDBObject.put("jobname", record.getJobname());
		basicDBObject.put("processname", record.getProcessname());
		basicDBObject.put("eventtime",	record.getEventtime());
		
		
		
		if (!basicDBObject.isEmpty())
			dbCollection.insert(basicDBObject);
	}
	
	
	
	public List<AlertBean> retriveAlert(AlertBean record) {	
		List<AlertBean> alertList = new ArrayList<AlertBean>();
		//serachQuery is the criteria clause
		BasicDBObject searchQuery = new BasicDBObject();
		if(record.getSubBusinessUnit() != null)
			searchQuery.put("subBusinessUnit", record.getSubBusinessUnit());
		if(record.getCategory() != null)
			searchQuery.put("category", record.getCategory());
		if(record.getAlertType() != null)
			searchQuery.put("alertType", record.getAlertType());
		if(record.getAlertDetail() != null)
			searchQuery.put("alertDetail", record.getAlertDetail());
		if(record.getApplicationName() != null)
			searchQuery.put("applicationName", record.getApplicationName());
		if(record.getApplicationName() != null)
			searchQuery.put("applicationFamily", record.getApplicationFamily());
		if(record.getJobname() != null)
			searchQuery.put("jobname", record.getJobname());
		if(record.getProcessname() != null)
			searchQuery.put("processname", record.getProcessname());
		if(record.getEventtime() != null)
			searchQuery.put("eventtime", record.getEventtime());
		
		
		DBCursor cursor = dbCollection.find(searchQuery);
		//keys are the values to get
		BasicDBObject keys=new BasicDBObject();
		keys.put("subBusinessUnit",1);
		keys.put("category",2);
		keys.put("alertType",3);
		keys.put("alertDetail",4);
		keys.put("applicationName",5);
		keys.put("applicationFamily",6);
		keys.put("tier",7);
		keys.put("node",8);
		keys.put("nativeBusinessUnit",9);
		keys.put("jobname",10);
		keys.put("processname",11);
		keys.put("eventtime",12);
		while (cursor.hasNext()) {
			BasicDBObject obj=(BasicDBObject)cursor.next();
			Date d =obj.getDate("eventtime");
			AlertBean e = new AlertBean(
					obj.getString("subBusinessUnit"),
					obj.getString("category"),
					obj.getString("alertType"),
					obj.getString("alertDetail"),
					obj.getString("applicationName"),
					obj.getString("applicationFamily"),
					obj.getString("tier"),
					obj.getString("node"),
					obj.getString("nativeBusinessUnit"),
					obj.getString("jobname"),
					obj.getString("processname"),
					new java.sql.Timestamp(d.getTime()));
			
			
			alertList.add(e);
					
		}
		return alertList;
	}
	
	
	public void updateAlert(AlertBean record, AlertBean criteria) {	
		
		//serachQuery is the criteria clause
		BasicDBObject searchQuery = new BasicDBObject();
		if(criteria.getSubBusinessUnit() != null)
			searchQuery.put("subBusinessUnit", criteria.getSubBusinessUnit());
		if(criteria.getCategory() != null)
			searchQuery.put("category", criteria.getCategory());
		if(criteria.getAlertType() != null)
			searchQuery.put("alertType", criteria.getAlertType());
		if(criteria.getAlertDetail() != null)
			searchQuery.put("alertDetail", criteria.getAlertDetail());
		if(criteria.getApplicationName() != null)
			searchQuery.put("applicationName", criteria.getApplicationName());
		if(criteria.getApplicationName() != null)
			searchQuery.put("applicationFamily", criteria.getApplicationFamily());
		
		if(criteria.getJobname() != null)
			searchQuery.put("jobname", criteria.getJobname());
		if(criteria.getProcessname() != null)
			searchQuery.put("processname", criteria.getProcessname());
		if(criteria.getEventtime() != null)
			searchQuery.put("eventtime", criteria.getEventtime());
		
		BasicDBObject basicDBObject = new BasicDBObject();
		basicDBObject.put("subBusinessUnit",
				record.getSubBusinessUnit());
		basicDBObject.put("category", record.getCategory());
		basicDBObject.put("alertType", record.getAlertType());
		basicDBObject.put("alertDetail", record.getAlertDetail());
		basicDBObject.put("applicationName",
				record.getApplicationName());
		basicDBObject.put("tier", record.getTier());
		basicDBObject.put("node", record.getNode());
		basicDBObject.put("nativeBusinessUnit",record.getNativeBusinessUnit());
		basicDBObject.put("jobname", record.getJobname());
		basicDBObject.put("processname", record.getProcessname());
		basicDBObject.put("eventtime",record.getEventtime());
		dbCollection.update(searchQuery, basicDBObject);
	}

}



import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;



import com.infy.gs.automation.beans.ErrorBean;
import com.infy.gs.automation.util.MongoConnection;
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;

public class ErrorDAO {


	DB db = MongoConnection.getConnection();
	DBCollection dbCollection = db.getCollection("logerror");
	
	
	
	public void createError(ErrorBean record) {		
		BasicDBObject basicDBObject = new BasicDBObject();
		
		basicDBObject.put("eventTime",
				record.getEventtime());
		basicDBObject.put("logFileName", record.getLogFileName());
		basicDBObject.put("level", record.getLevel());
		basicDBObject.put("logger", record.getLogger());
		basicDBObject.put("logthread", record.getLogthread());
		basicDBObject.put("logmessage", record.getLogmessage());
		
			
		if (!basicDBObject.isEmpty())
			dbCollection.insert(basicDBObject);
	}
	
	
	
	public List<ErrorBean> retriveError(ErrorBean record) {	
		List<ErrorBean> errorList = new ArrayList<ErrorBean>();
		//serachQuery is the criteria clause
		BasicDBObject searchQuery = new BasicDBObject();
		
		if(record.getEventtime() != null)
			searchQuery.put("eventTime", record.getEventtime());
		if(record.getLogFileName() != null)
			searchQuery.put("logFileName", record.getLogFileName());
		if(record.getLogger() != null)
			searchQuery.put("logger", record.getLogger());
		if(record.getLogmessage()!= null)
			searchQuery.put("logmessage", record.getLogmessage());
				
		
		DBCursor cursor = dbCollection.find(searchQuery);
		//keys are the values to get
		BasicDBObject keys=new BasicDBObject();
		keys.put("eventTime",1);
		keys.put("logFileName",2);
		keys.put("logger",3);
		keys.put("logmessage",4);
		
		while (cursor.hasNext()) {
			BasicDBObject obj=(BasicDBObject)cursor.next();
			
			
			Date d =new Date();
			d=obj.getDate("eventTime");
			
		
			java.sql.Timestamp sq = new java.sql.Timestamp(d.getTime());
			ErrorBean e = new ErrorBean(
					sq,
					obj.getString("logFileName"),
					obj.getString("level"),
					obj.getString("logger"),obj.getString("logthread"),obj.getString("logmessage"));
			
			
			errorList.add(e);
					
		}
		return errorList;
	}
	
	
	public void updateError(ErrorBean record, ErrorBean criteria) {	
		
		//serachQuery is the criteria clause
		BasicDBObject searchQuery = new BasicDBObject();
		
		
		
		if(criteria.getEventtime() != null)
			searchQuery.put("eventTime", criteria.getEventtime());
		if(criteria.getLogFileName() != null)
			searchQuery.put("logFileName", criteria.getLogFileName());
		if(criteria.getLogger()!= null)
			searchQuery.put("logger", criteria.getLogger());
		if(criteria.getLogmessage() != null)
			searchQuery.put("logmessage", criteria.getLogmessage());
		
		
		BasicDBObject basicDBObject = new BasicDBObject();
		basicDBObject.put("eventTime",
				record.getEventtime());
		basicDBObject.put("logFileName", record.getLogFileName());
		basicDBObject.put("logger", record.getLogger());
		basicDBObject.put("logmessage", record.getLogmessage());
		
		dbCollection.update(searchQuery, basicDBObject);
	}
	
	
}



import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.List;

import com.infy.gs.automation.beans.FeatureExtractionBean;
import com.infy.gs.automation.util.MongoConnection;
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;

public class FeatureExtractionDAO {

		
	DB db = MongoConnection.getConnection();
	DBCollection dbCollection = db.getCollection("featureextraction");
	
	public void createExtraction(FeatureExtractionBean record) {		
		BasicDBObject basicDBObject = new BasicDBObject();
		basicDBObject.put("processName",
				record.getProcessName());
		basicDBObject.put("eventTime", record.getEventTime());
		basicDBObject.put("error", record.getError());
		basicDBObject.put("alertDetail", record.getAlertDetail());
		basicDBObject.put("alertCategory", record.getAlertCategory());
		
		
		
		
		if (!basicDBObject.isEmpty())
			dbCollection.insert(basicDBObject);
	}
	
	
	
	public List<FeatureExtractionBean> retriveExtraction(FeatureExtractionBean record) {	
		List<FeatureExtractionBean> featureList = new ArrayList<FeatureExtractionBean>();
		//serachQuery is the criteria clause
		BasicDBObject searchQuery = new BasicDBObject();
		if(record.getProcessName() != null)
			searchQuery.put("processName", record.getProcessName());
		if(record.getEventTime()!= null)
			searchQuery.put("eventTime", record.getEventTime());
		if(record.getError() != null)
			searchQuery.put("error", record.getError());
		if(record.getAlertDetail() != null)
			searchQuery.put("alertDetail", record.getAlertDetail());
		if(record.getAlertCategory() != null)
			searchQuery.put("alertCategory", record.getAlertCategory());
		
		
		
		DBCursor cursor = dbCollection.find(searchQuery);
		//keys are the values to get
		BasicDBObject keys=new BasicDBObject();
		keys.put("processName",1);
		keys.put("eventTime",2);
		keys.put("error",3);
		keys.put("alertDetail",4);
		keys.put("alertCategory",5);
		
		while (cursor.hasNext()) {
			BasicDBObject obj=(BasicDBObject)cursor.next();
			FeatureExtractionBean e = new FeatureExtractionBean(
					obj.getString("processName"),
					(Timestamp) obj.getDate("eventtime"),
					obj.getString("error"),
					obj.getString("alertDetail"),
					obj.getString("alertCategory")
				
					);
			
			
			featureList.add(e);
					
		}
		return featureList;
	}
	
	
	public void updateExtraction(FeatureExtractionBean record, FeatureExtractionBean criteria) {	
		
		//serachQuery is the criteria clause
		BasicDBObject searchQuery = new BasicDBObject();
		if(criteria.getProcessName() != null)
			searchQuery.put("processName", criteria.getProcessName());
		if(criteria.getEventTime() != null)
			searchQuery.put("eventTime", criteria.getEventTime());
		if(criteria.getError() != null)
			searchQuery.put("error", criteria.getError());
		if(criteria.getAlertDetail() != null)
			searchQuery.put("alertDetail", criteria.getAlertDetail());
		if(criteria.getAlertCategory() != null)
			searchQuery.put("alertCategory", criteria.getAlertCategory());
		
		BasicDBObject basicDBObject = new BasicDBObject();
		basicDBObject.put("processName",record.getProcessName());
		basicDBObject.put("eventTime", record.getEventTime());
		basicDBObject.put("error", record.getError());
		basicDBObject.put("alertDetail", record.getAlertDetail());
		basicDBObject.put("alertCategory",record.getAlertCategory());
		
		dbCollection.update(searchQuery, basicDBObject);
	}

}



import java.util.ArrayList;
import java.util.List;

import com.infy.gs.automation.beans.ProcessBean;
import com.infy.gs.automation.util.MongoConnection;
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;

public class ProcessDAO {

	DB db = MongoConnection.getConnection();
	DBCollection dbCollection = db.getCollection("process");
	
	public void createProcess(ProcessBean record) {		
		BasicDBObject basicDBObject = new BasicDBObject();
		basicDBObject.put("processName",
				record.getProcessName());
		basicDBObject.put("logFileName", record.getLogFileName());
		
		if (!basicDBObject.isEmpty())
			dbCollection.insert(basicDBObject);
	}
	
	
	
	public List<ProcessBean> retriveProcess(ProcessBean record) {	
		List<ProcessBean> processList = new ArrayList<ProcessBean>();
		//serachQuery is the criteria clause
		BasicDBObject searchQuery = new BasicDBObject();
		if(record.getProcessName() != null)
			searchQuery.put("processName", record.getProcessName());
		if(record.getLogFileName() != null)
			searchQuery.put("logFileName", record.getLogFileName());
		
		
		DBCursor cursor = dbCollection.find(searchQuery);
		//keys are the values to get
		BasicDBObject keys=new BasicDBObject();
		keys.put("processName",1);
		keys.put("logFileName",2);
		
		while (cursor.hasNext()) {
			BasicDBObject obj=(BasicDBObject)cursor.next();
			ProcessBean e = new ProcessBean(
					obj.getString("processName"),
					obj.getString("logFileName"));
			
			
			processList.add(e);
					
		}
		return processList;
	}
	
	
	public void updateProcess(ProcessBean record, ProcessBean criteria) {	
		
		//serachQuery is the criteria clause
		BasicDBObject searchQuery = new BasicDBObject();
		if(criteria.getProcessName() != null)
			searchQuery.put("processName", criteria.getProcessName());
		if(criteria.getLogFileName() != null)
			searchQuery.put("logFileName", criteria.getLogFileName());
		
		BasicDBObject basicDBObject = new BasicDBObject();
		basicDBObject.put("processName",
				record.getProcessName());
		basicDBObject.put("logFileName", record.getLogFileName());
		
		dbCollection.update(searchQuery, basicDBObject);
	}

}



import java.net.UnknownHostException;

import com.mongodb.DB;
import com.mongodb.MongoClient;
import com.mongodb.MongoClientOptions;

public class MongoConnection {
	
	private static final String MongoHost = "localhost";
	private static final int MongoPort = 27017;
	
	public static DB getConnection() {
		 MongoClient mongoClient;
		try {
			mongoClient = new MongoClient( MongoHost , MongoClientOptions.builder().connectionsPerHost(1).build());
			System.out.println("DB connected -----------------------");
			DB db = mongoClient.getDB("sparkdb");
			return db;
		} catch (UnknownHostException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			return null;
		}	
		// MongoClient mongo = new MongoClient( "localhost" , 27017 );
		 
		 //boolean auth = db.authenticate("username", "password".toCharArray());
		
	}
	
	
	
}


import java.io.Serializable;
import java.sql.Timestamp;

public class AlertBean implements Serializable{

	private String subBusinessUnit;
	private String category;
	private String alertType;
	private String alertDetail;
	private String applicationName;
	private String applicationFamily;
	private String tier;
	private String node;
	private String nativeBusinessUnit; private String jobname;
	private String processname; 
	private Timestamp eventtime;
	
	
	public String getSubBusinessUnit() {
		return subBusinessUnit;
	}
	public void setSubBusinessUnit(String subBusinessUnit) {
		this.subBusinessUnit = subBusinessUnit;
	}
	public String getCategory() {
		return category;
	}
	public void setCategory(String category) {
		this.category = category;
	}
	public String getAlertType() {
		return alertType;
	}
	public void setAlertType(String alertType) {
		this.alertType = alertType;
	}
	public String getAlertDetail() {
		return alertDetail;
	}
	public void setAlertDetail(String alertDetail) {
		this.alertDetail = alertDetail;
	}
	public String getApplicationName() {
		return applicationName;
	}
	public void setApplicationName(String applicationName) {
		this.applicationName = applicationName;
	}
	
	
	public String getApplicationFamily() {
		return applicationFamily;
	}
	public void setApplicationFamily(String applicationFamily) {
		this.applicationFamily = applicationFamily;
	}
	public String getTier() {
		return tier;
	}
	public void setTier(String tier) {
		this.tier = tier;
	}
	public String getNode() {
		return node;
	}
	public void setNode(String node) {
		this.node = node;
	}
	public String getNativeBusinessUnit() {
		return nativeBusinessUnit;
	}
	public void setNativeBusinessUnit(String nativeBusinessUnit) {
		this.nativeBusinessUnit = nativeBusinessUnit;
	}
	
	
	
	public String getJobname() {
		return jobname;
	}
	public void setJobname(String jobname) {
		this.jobname = jobname;
	}
	public String getProcessname() {
		return processname;
	}
	public void setProcessname(String processname) {
		this.processname = processname;
	}
	public Timestamp getEventtime() {
		return eventtime;
	}
	public void setEventtime(Timestamp eventtime) {
		this.eventtime = eventtime;
	}
	public AlertBean(){
		
	}
	
	public AlertBean(String subBusinessUnit, String category, String alertType,
			String alertDetail, String applicationName,
			String applicationFamily, String tier, String node,
			String nativeBusinessUnit, String jobname, String processname,
			Timestamp eventtime) {
		super();
		this.subBusinessUnit = subBusinessUnit;
		this.category = category;
		this.alertType = alertType;
		this.alertDetail = alertDetail;
		this.applicationName = applicationName;
		this.applicationFamily = applicationFamily;
		this.tier = tier;
		this.node = node;
		this.nativeBusinessUnit = nativeBusinessUnit;
		this.jobname = jobname;
		this.processname = processname;
		this.eventtime = eventtime;
	}
	
	
	
	
}


import java.io.Serializable;
import java.sql.Timestamp;

public class ErrorBean implements Serializable {
private Timestamp eventtime;
private String logFileName;
private String level;
private String logger;
private String logthread;
private String logmessage;
public Timestamp getEventtime() {
	return eventtime;
}
public void setEventtime(Timestamp eventtime) {
	this.eventtime = eventtime;
}
public String getLogFileName() {
	return logFileName;
}
public void setLogFileName(String logFileName) {
	this.logFileName = logFileName;
}
public String getLevel() {
	return level;
}
public void setLevel(String level) {
	this.level = level;
}
public String getLogger() {
	return logger;
}
public void setLogger(String logger) {
	this.logger = logger;
}
public String getLogthread() {
	return logthread;
}
public void setLogthread(String logthread) {
	this.logthread = logthread;
}
public String getLogmessage() {
	return logmessage;
}
public void setLogmessage(String logmessage) {
	this.logmessage = logmessage;
}

public ErrorBean(){
	
}

public ErrorBean(Timestamp eventtime, String logFileName, String level,
		String logger, String logthread, String logmessage) {
	super();
	this.eventtime = eventtime;
	this.logFileName = logFileName;
	this.level = level;
	this.logger = logger;
	this.logthread = logthread;
	this.logmessage = logmessage;
}




}

import java.io.Serializable;
import java.sql.Timestamp;

public class FeatureExtractionBean implements Serializable {
	private String processName;
	private Timestamp eventTime;
	private String error;
	private String alertDetail;
	private String alertCategory;
	
	public String getProcessName() {
		return processName;
	}
	public void setProcessName(String processName) {
		this.processName = processName;
	}
	public Timestamp getEventTime() {
		return eventTime;
	}
	public void setEventTime(Timestamp eventTime) {
		this.eventTime = eventTime;
	}
	public String getError() {
		return error;
	}
	public void setError(String error) {
		this.error = error;
	}
	public String getAlertDetail() {
		return alertDetail;
	}
	public void setAlertDetail(String alertDetail) {
		this.alertDetail = alertDetail;
	}
	public String getAlertCategory() {
		return alertCategory;
	}
	public void setAlertCategory(String alertCategory) {
		this.alertCategory = alertCategory;
	}
	public FeatureExtractionBean(String processName, Timestamp eventTime,
			String error, String alertDetail, String alertCategory) {
		
		this.processName = processName;
		this.eventTime = eventTime;
		this.error = error;
		this.alertDetail = alertDetail;
		this.alertCategory = alertCategory;
	}
	
	

}


import java.io.Serializable;
import java.sql.Timestamp;

public class JoinKeyBean implements Serializable{

	private Timestamp eventtime;
	private String processName;
	
	public JoinKeyBean(){
		
	}

	public Timestamp getEventtime() {
		return eventtime;
	}

	public void setEventtime(Timestamp eventtime) {
		this.eventtime = eventtime;
	}

	public String getProcessName() {
		return processName;
	}

	public void setProcessName(String processName) {
		this.processName = processName;
	}

	public JoinKeyBean(Timestamp eventtime, String processName) {
		super();
		this.eventtime = eventtime;
		this.processName = processName;
	}

	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result
				+ ((eventtime == null) ? 0 : eventtime.hashCode());
		result = prime * result
				+ ((processName == null) ? 0 : processName.hashCode());
		return result;
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		JoinKeyBean other = (JoinKeyBean) obj;
		if (eventtime == null) {
			if (other.eventtime != null)
				return false;
		} else if (!eventtime.equals(other.eventtime))
			return false;
		if (processName == null) {
			if (other.processName != null)
				return false;
		} else if (!processName.equals(other.processName))
			return false;
		return true;
	}
	
	 
	
	
	
		/* @Override
	        public boolean equals(Object obj) {
	            if (obj == this)
	                return true;
	            if (!(obj instanceof JoinKeyBean))
	                return false;
	            JoinKeyBean bean = (JoinKeyBean) obj;
	            return (bean.getEventtime().equals(this.getEventtime()) && bean.getProcessName().equals(this.getProcessName()));
	        }*/
	
		/* @Override
	        public int hashCode() {
	            result=31*result+(processName!=null ? processName.hashCode():0);
	            return result;
	        }*/
	
}



import java.io.Serializable;
import java.sql.Timestamp;
import java.util.Date;

import org.apache.commons.io.FilenameUtils;
import org.apache.commons.io.IOUtils;
import org.apache.log4j.Level;
import org.apache.log4j.spi.LoggingEvent;

/**
 * A simple facade over the top of the log4j LoggingEvent to clean up stuff like
 * the stack trace and the timestamp
 */
public class LogData implements Serializable{
    private  String levelPattren;
    private  Timestamp timestamp;
    private  String loggerName;
    private  String threadName;
    private  String message;
    private  String stackTrace;
    private  String logfileName;

	
	public void setLoggerName(String loggerName) {
		this.loggerName = loggerName;
	}
	public void setThreadName(String threadName) {
		this.threadName = threadName;
	}
	public void setMessage(String message) {
		this.message = message;
	}
	public void setStackTrace(String stackTrace) {
		this.stackTrace = stackTrace;
	}
	public void setLogfileName(String logfileName) {
		this.logfileName = logfileName;
	}
	
	public String getLevelPattren() {
		return levelPattren;
	}
	public void setLevelPattren(String levelPattren) {
		this.levelPattren = levelPattren;
	}
	
	public Timestamp getTimestamp() {
		return timestamp;
	}
	public void setTimestamp(Timestamp timestamp) {
		this.timestamp = timestamp;
	}
	public String getLoggerName() {
		return loggerName;
	}
	public String getThreadName() {
		return threadName;
	}
	public String getMessage() {
		return message;
	}
	public String getStackTrace() {
		return stackTrace;
	}
	public String getLogfileName() {
		return logfileName;
	}
    

}


import java.io.Serializable;

public class ProcessBean implements Serializable{
	private String processName; 
	private String logFileName;
	
	public String getProcessName() {
		return processName;
	}
	public void setProcessName(String processName) {
		this.processName = processName;
	}
	public String getLogFileName() {
		return logFileName;
	}
	public void setLogFileName(String logFileName) {
		this.logFileName = logFileName;
	}
	public ProcessBean(String processName, String logFileName) {
		super();
		this.processName = processName;
		this.logFileName = logFileName;
	}
	public ProcessBean() {
		
	}
	
	
}
------------------------------------Kafka------
package com.infy.spark.utilities;

import java.util.LinkedHashMap;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class LogPattternMatch {

	private Map<String,Integer> exceptionCountMap = new LinkedHashMap<String,Integer>();

	public Map<String, Integer> matchAndCount(String line) {

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		while (matcher.find()){

			exceptionName = matcher.group();
			incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println(exceptionName);
		}
		return exceptionCountMap;
	}
	
	public Boolean isExceptionMatch(String line) {
		
		Boolean isMAtch = false;

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		while (matcher.find()){

			exceptionName = matcher.group();
			incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println("______________________________________isExceptionMathch "+exceptionName);
			isMAtch = true;
		}
		//exceptionCountMap.
		return isMAtch;
	}

public String searchException(String line) {
		
		//Boolean isMAtch = false;

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		if(matcher.find()){

			exceptionName = matcher.group();
		//	incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println("Search exception.....................................  "+exceptionName);
		}
		//exceptionCountMap.
		return exceptionName;
	}
	
	public static void main(String[] args) {

		/*String text = "\"Exception in thread \"main\" java.lang.NullPointerException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);"+
				" Exception in thread \"main\" java.lang.ClassCastException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);"+
				" Exception in thread \"main\" java.lang.ClassCastException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);";
*/
		String text = " java.lang.NullPointerException";
		new LogPattternMatch().matchAndCount(text);
	}

	private static void incrementMapCount(Map<String,Integer> map, String exceptionName) {

		Integer currentCount = (Integer)map.get(exceptionName);

		if (currentCount == null) {

			map.put(exceptionName, new Integer(1));
		}
		else {

			currentCount = new Integer(currentCount.intValue() + 1);
			map.put(exceptionName,currentCount);
		}
	}
}

package com.infy.gs.automation.producer;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.Reader;
import java.util.Iterator;
import java.util.Map;
import java.util.NoSuchElementException;
import java.util.Properties;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;

import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVRecord;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;

public class AlertProducerService {
	private String fileName;
	private String colHeader;
	private Producer<String, String> producer;

	/**
	 * Initialize Kafka configuration
	 */
	public void initKafkaConfig() {

		// Build the configuration required for connecting to Kafka
		Properties props = new Properties();
		// List of Kafka brokers. If there're multiple brokers, they're
		props.put("metadata.broker.list", "localhost:9092");
		// Serializer used for sending data to kafka. Since we are sending
		// string, we are using StringEncoder.
		props.put("serializer.class", "kafka.serializer.StringEncoder");
		// We want acks from Kafka that messages are properly received.
		props.put("request.required.acks", "1");

		// Create the producer instance
		ProducerConfig config = new ProducerConfig(props);
		producer = new Producer<String, String>(config);
	}

	/**
	 * Initialize configuration for file to be read (csv)
	 * 
	 * @param fileName
	 * @throws IOException
	 */
	public void initFileConfig(String fileName) throws IOException, Exception {
		this.fileName = fileName;

	
		try {
			
			
			FileInputStream fis = new FileInputStream(fileName);
					  
		
			InputStream inStream =fis;
			Reader reader = new InputStreamReader(inStream);
			BufferedReader buffReader = IOUtils.toBufferedReader(reader);

			// Get the header line to initialize CSV parser
			colHeader = buffReader.readLine();
			System.out.println("File header :: " + colHeader);

			if (StringUtils.isEmpty(colHeader)) {
				throw new Exception("Column header is null, something is wrong");
			}
		} catch (IOException e) {
			System.out.println(e.getMessage());
			throw e;
		}
		
	}

	/**
	 * Send csv file data to the named topic on Kafka broker
	 * 
	 * @param topic
	 * @throws IOException
	 */
	public void sendFileDataToKafka(String topic) throws IOException {

		Iterable<CSVRecord> csvRecords = null;

		// Parse the CSV file, using the column header
		
		try {
						
			FileInputStream fis = new FileInputStream(fileName);
			  
			
			InputStream inStream =fis;
			
			
			Reader reader = new InputStreamReader(inStream);

			String[] colNames = StringUtils.split(colHeader, ',');
			csvRecords = CSVFormat.DEFAULT.withIgnoreEmptyLines(true).withHeader(colNames).parse(reader);
			
		} catch (IOException e) {
			System.out.println(e);
			throw e;
		}
		
		// We iterate over the records and send each over to Kafka broker
		// Get the next record from input file
		CSVRecord csvRecord = null;
		Iterator<CSVRecord> csvRecordItr = csvRecords.iterator();
		boolean firstRecDone = false;
		
		while (csvRecordItr.hasNext()) {
			try {
				csvRecord = csvRecordItr.next();
				
				
				if (!firstRecDone) {
					firstRecDone = true;
					continue;
				}
				// Get a map of column name and value for a record
				Map<String, String> keyValueRecord = csvRecord.toMap();

				// Create the message to be sent
				String message = "";
				int size = keyValueRecord.size();
				int count = 0;
				String msgstr[]=null;
				for (String key : keyValueRecord.keySet()) {
					count++;
					
					
					message = message + StringUtils.replace(key, "\"", "") + "=";
					
					if (keyValueRecord.get(key).contains(",") || keyValueRecord.get(key).contains("=") || keyValueRecord.get(key).isEmpty()){
					
					 msgstr = StringUtils.replace(keyValueRecord.get(key),"\"", "").replace("=", ":").replaceAll(",,", ",No data,").split(",(?=([^\"]*\"[^\"]*\")*[^\"]*$)");
					
					 for(int i=0;i<msgstr.length;i++){
					 message =message +" " + msgstr[i];
					 }
					}
					else
						message =message +StringUtils.replace(keyValueRecord.get(key),"\"", "");
					
					if (count != size) {
						message = message + ",";
					}
				}
			//	String msg = remCommaFmData(message);
				// Send the message
			//	System.out.println(msg);
				System.out.println(message+"*****************************");
				KeyedMessage<String, String> data = new KeyedMessage<String, String>(
						topic,  message);
				producer.send(data);

			} catch (NoSuchElementException e) {
				System.out.println(e.getMessage());
			}
		}
		
	}

	/**
	 * Cleanup stuff
	 */
	public void cleanup() {
		producer.close();
	}
	
}


-----------Log kafka

package com.infy.gs.automation.log.parser;

import java.io.File;

import org.apache.commons.lang.Validate;

public class AppenderConfig {
    private final String name;
    private final String pattern;
    private final File logFile;
    
    public AppenderConfig(final String name, final String pattern, final File logFile) {
        Validate.notNull(name);
        Validate.notNull(pattern);
        Validate.notNull(logFile);
        
        this.name = name;
        this.pattern = pattern;
        this.logFile = logFile;
    }

    public File getLogFile() {
        return logFile;
    }

    public String getName() {
        return name;
    }

    public String getPattern() {
        return pattern;
    }
}



package com.infy.gs.automation.log.parser;

public enum Level {
    FATAL, ERROR, WARN, INFO, DEBUG, TRACE;
}

package com.infy.gs.automation.log.parser;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ListLogDataHandler implements LogDataHandler {
    private final List<LogData> logList = 
        Collections.synchronizedList(new ArrayList<LogData>());
    
    @Override
    public void handle(LogData logData) {
        logList.add(logData);
    }

    public int getSize() {
        return logList.size();
    }
}



package com.infy.gs.automation.log.parser;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Parse properties and provide access to any appender objects.
 */
public class Log4jAppenderConfigLoader {
    private Logger logger = LoggerFactory.getLogger(getClass());
    
    private static final String FILE_PROP = "file";
    private static final String PATTERN_PROP = "conversion";
    
    private final List<AppenderConfig> appenderList = new ArrayList<AppenderConfig>();
    
    public Log4jAppenderConfigLoader(File propsFile) throws IOException {
        this(new FileInputStream(propsFile));
    }
    
    public Log4jAppenderConfigLoader(InputStream inputStream) throws IOException {
        Map<String, Map<String, String>> appenderMap = 
            LogFilePatternLayoutBuilder.getPropertiesFileAppenderConfiguration(inputStream);
        
        for (Map.Entry<String, Map<String, String>> appenderEntry : appenderMap.entrySet()) {
            String name = appenderEntry.getKey();
            String pattern = appenderEntry.getValue().get(PATTERN_PROP);
            File file = new File(appenderEntry.getValue().get(FILE_PROP));
            
            logger.debug("File Appender {} [Pattern: {}, File: {}]", new Object[]{name, pattern, file});
            
            appenderList.add(new AppenderConfig(name, pattern, file));
        }   
    }
    
    /**
     * A convenience method to get the appender for a particular file.  This is based
     * on just the filename, not the complete path of the file.
     * 
     * @param file
     * @return
     */
    public AppenderConfig getAppenderConfig(File file) {
        for(AppenderConfig appenderConfig : appenderList) {
        	System.out.println(file.getName() +"-------------------"+ appenderConfig.getLogFile().getName());
            if (file.getName().equals(appenderConfig.getLogFile().getName())) {
                return appenderConfig;
            }
        }
        throw new IllegalArgumentException("Appender Config not found: " + file.getName());
    }
}


package com.infy.gs.automation.log.parser;

import java.io.Serializable;
import java.util.Date;

import org.apache.commons.io.IOUtils;
import org.apache.log4j.spi.LoggingEvent;

/**
 * A simple facade over the top of the log4j LoggingEvent to clean up stuff like
 * the stack trace and the timestamp
 */
public class LogData implements Serializable{
    private  Level level;
    private  Date timestamp;
    private  String loggerName;
    private  String threadName;
    private String message;
    private  String stackTrace;
    private  String logfileName;
    
    public LogData(){
    	
    }

	public LogData(LoggingEvent loggingEvent) {
        level = Level.valueOf(loggingEvent.getLevel().toString());
        timestamp = new Date(loggingEvent.getTimeStamp());
        loggerName = loggingEvent.getLoggerName();
        threadName = loggingEvent.getThreadName();
        message = loggingEvent.getRenderedMessage();
 
        StringBuilder builder = new StringBuilder();
        for (String st : loggingEvent.getThrowableStrRep()) {
            builder.append(st);
            builder.append(IOUtils.LINE_SEPARATOR);
        }
        stackTrace = builder.toString();
    }

	public Level getLevel() {
		return level;
	}

	public void setLevel(Level level) {
		this.level = level;
	}

	public Date getTimestamp() {
		return timestamp;
	}

	public void setTimestamp(Date timestamp) {
		this.timestamp = timestamp;
	}

	public String getLoggerName() {
		return loggerName;
	}

	public void setLoggerName(String loggerName) {
		this.loggerName = loggerName;
	}

	public String getThreadName() {
		return threadName;
	}

	public void setThreadName(String threadName) {
		this.threadName = threadName;
	}

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}

	public String getStackTrace() {
		return stackTrace;
	}

	public void setStackTrace(String stackTrace) {
		this.stackTrace = stackTrace;
	}

	public String getLogfileName() {
		return logfileName;
	}

	public void setLogfileName(String logfileName) {
		this.logfileName = logfileName;
	}

 
}


package com.infy.gs.automation.log.parser;


public interface LogDataHandler {
    void handle(LogData logData);
}


package com.infy.gs.automation.log.parser;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

import org.apache.log4j.spi.LoggingEvent;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogFileParser extends LogFilePatternReceiver {
    private Logger logger = LoggerFactory.getLogger(getClass());
    private final LogDataHandler collector;
    
    public LogFileParser(final AppenderConfig appenderConfig, 
                                        final LogDataHandler collector) {
        
        String pattern = LogFilePatternLayoutBuilder.getLogFormatFromPatternLayout(appenderConfig.getPattern());
        logger.debug("Pattern {}", pattern);
        
        String timeStampFormat = LogFilePatternLayoutBuilder.getTimeStampFormat(appenderConfig.getPattern());
        logger.debug("Time Stamp Format {}", timeStampFormat);
        
        setTimestampFormat(timeStampFormat);
        setLogFormat(pattern);
        initialize();
        createPattern();
        
        this.collector = collector;
    }
    
    public void process(InputStream inputStream) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
        super.process(reader);
    }
    
    @Override
    public void doPost(LoggingEvent event) {
        collector.handle(new LogData(event));
    }
}


package com.infy.gs.automation.log.parser;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Locale;
import java.util.Map;
import java.util.Properties;

import org.apache.log4j.helpers.OptionConverter;
import org.apache.log4j.pattern.ClassNamePatternConverter;
import org.apache.log4j.pattern.DatePatternConverter;
import org.apache.log4j.pattern.FileLocationPatternConverter;
import org.apache.log4j.pattern.FullLocationPatternConverter;
import org.apache.log4j.pattern.LevelPatternConverter;
import org.apache.log4j.pattern.LineLocationPatternConverter;
import org.apache.log4j.pattern.LineSeparatorPatternConverter;
import org.apache.log4j.pattern.LiteralPatternConverter;
import org.apache.log4j.pattern.LoggerPatternConverter;
import org.apache.log4j.pattern.LoggingEventPatternConverter;
import org.apache.log4j.pattern.MessagePatternConverter;
import org.apache.log4j.pattern.MethodLocationPatternConverter;
import org.apache.log4j.pattern.NDCPatternConverter;
import org.apache.log4j.pattern.PatternParser;
import org.apache.log4j.pattern.PropertiesPatternConverter;
import org.apache.log4j.pattern.RelativeTimePatternConverter;
import org.apache.log4j.pattern.SequenceNumberPatternConverter;
import org.apache.log4j.pattern.ThreadPatternConverter;

public class LogFilePatternLayoutBuilder {
	public static String getLogFormatFromPatternLayout(String patternLayout) {
		String input = OptionConverter.convertSpecialChars(patternLayout);
		List converters = new ArrayList();
		List fields = new ArrayList();
		Map converterRegistry = null;

		PatternParser.parse(input, converters, fields, converterRegistry,
				PatternParser.getPatternLayoutRules());
		return getFormatFromConverters(converters);
	}

	public static String getTimeStampFormat(String patternLayout) {
		int basicIndex = patternLayout.indexOf("%d");
		if (basicIndex < 0) {
			return null;
		}

		int index = patternLayout.indexOf("%d{");
		// %d - default
		if (index < 0) {
			return "yyyy-MM-dd HH:mm:ss,SSS";
		}

		int length = patternLayout.substring(index).indexOf("}");
		String timestampFormat = patternLayout.substring(
				index + "%d{".length(), index + length);
		if (timestampFormat.equals("ABSOLUTE")) {
			return "HH:mm:ss,SSS";
		}
		if (timestampFormat.equals("ISO8601")) {
			return "yyyy-MM-dd HH:mm:ss,SSS";
		}
		if (timestampFormat.equals("DATE")) {
			return "dd MMM yyyy HH:mm:ss,SSS";
		}
		return timestampFormat;
	}

	private static String getFormatFromConverters(List converters) {
		StringBuffer buffer = new StringBuffer();
		for (Iterator iter = converters.iterator(); iter.hasNext();) {
			LoggingEventPatternConverter converter = (LoggingEventPatternConverter) iter
					.next();
			if (converter instanceof DatePatternConverter) {
				buffer.append("TIMESTAMP");
			} else if (converter instanceof MessagePatternConverter) {
				buffer.append("MESSAGE");
			} else if (converter instanceof LoggerPatternConverter) {
				buffer.append("LOGGER");
			} else if (converter instanceof ClassNamePatternConverter) {
				buffer.append("CLASS");
			} else if (converter instanceof RelativeTimePatternConverter) {
				buffer.append("PROP(RELATIVETIME)");
			} else if (converter instanceof ThreadPatternConverter) {
				buffer.append("THREAD");
			} else if (converter instanceof NDCPatternConverter) {
				buffer.append("NDC");
			} else if (converter instanceof LiteralPatternConverter) {
				LiteralPatternConverter literal = (LiteralPatternConverter) converter;
				// format shouldn't normally take a null, but we're getting a
				// literal, so passing in the
				// buffer will work
				literal.format(null, buffer);
			} else if (converter instanceof SequenceNumberPatternConverter) {
				buffer.append("PROP(log4jid)");
			} else if (converter instanceof LevelPatternConverter) {
				buffer.append("LEVEL");
			} else if (converter instanceof MethodLocationPatternConverter) {
				buffer.append("METHOD");
			} else if (converter instanceof FullLocationPatternConverter) {
				buffer.append("PROP(locationInfo)");
			} else if (converter instanceof LineLocationPatternConverter) {
				buffer.append("LINE");
			} else if (converter instanceof FileLocationPatternConverter) {
				buffer.append("FILE");
			} else if (converter instanceof PropertiesPatternConverter) {
				// PropertiesPatternConverter propertiesConverter =
				// (PropertiesPatternConverter) converter;
				// String option = propertiesConverter.getOption();
				// if (option != null && option.length() > 0) {
				// buffer.append("PROP(" + option + ")");
				// } else {
				buffer.append("PROP(PROPERTIES)");
				// }
			} else if (converter instanceof LineSeparatorPatternConverter) {
				// done
			}
		}
		return buffer.toString();
	}

	public static Map<String, Map<String, String>> getPropertiesFileAppenderConfiguration(
			File propertyFile) throws IOException {
		return getPropertiesFileAppenderConfiguration(new FileInputStream(
				propertyFile));
	}

	public static Map<String, Map<String, String>> getPropertiesFileAppenderConfiguration(
			InputStream inputStream) throws IOException {
		Map<String, Map<String, String>> result = new HashMap<String, Map<String, String>>();
		String appenderPrefix = "log4j.appender";
		Properties props = new Properties();
		try {
			props.load(inputStream);
			Map<String, String> appenders = new HashMap<String, String>();
			for (String propertyName : props.stringPropertyNames()) {
				if (propertyName.startsWith(appenderPrefix)) {
					String value = propertyName.substring(appenderPrefix
							.length() + 1);
					if (value.indexOf(".") == -1) {
						// no sub-values - this entry is the appender name &
						// class
						appenders.put(value, props.getProperty(propertyName)
								.trim());
						break;
					}
				}
			}

			for (Map.Entry<String, String> appenderEntry : appenders.entrySet()) {
				String appenderName = appenderEntry.getKey();
				String conversion = props.getProperty(appenderPrefix + "."
						+ appenderName + ".layout.ConversionPattern");
				String file = props.getProperty(appenderPrefix + "."
						+ appenderName + ".File");
				if (conversion != null && file != null) {
					Map<String, String> entry = new HashMap<String, String>();
					entry.put("file", file.trim());
					entry.put("conversion", conversion.trim());
					result.put(appenderName, entry);
				}

			}

			/*
			 * example: log4j.appender.R=org.apache.log4j.RollingFileAppender
			 * log4j.appender.R.File=${catalina.base}/logs/tomcat.log
			 * log4j.appender.R.MaxFileSize=10MB
			 * log4j.appender.R.MaxBackupIndex=10
			 * log4j.appender.R.layout=org.apache.log4j.PatternLayout
			 * log4j.appender.R.layout.ConversionPattern=%d - %p %t %c - %m%n
			 */
		} catch (IOException ioe) {
		} finally {
			if (inputStream != null) {
				inputStream.close();
			}
		}
		// don't return null
		return result;
	}
}


package com.infy.gs.automation.log.parser;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.StringTokenizer;
import java.util.regex.MatchResult;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.util.regex.PatternSyntaxException;

import org.apache.log4j.Level;
import org.apache.log4j.Logger;
import org.apache.log4j.spi.LocationInfo;
import org.apache.log4j.spi.LoggingEvent;
import org.apache.log4j.spi.ThrowableInformation;
import org.slf4j.LoggerFactory;

/**
 * - tailing may fail if the file rolls over.
 * <p>
 * <b>Example receiver configuration settings</b> (add these as params, specifying a LogFilePatternReceiver
 * 'plugin'):<br>
 * param: "timestampFormat" value="yyyy-MM-d HH:mm:ss,SSS"<br>
 * param: "logFormat" value="PROP(RELATIVETIME) [THREAD] LEVEL LOGGER * - MESSAGE"<br>
 * param: "fileURL" value="file:///c:/events.log"<br>
 * param: "tailing" value="true"
 * <p>
 * 
 */
public abstract class LogFilePatternReceiver {
    private org.slf4j.Logger logger = LoggerFactory.getLogger(getClass());
    
    private final List keywords = new ArrayList();

    private static final String PROP_START = "PROP(";
    private static final String PROP_END = ")";

    private static final String LOGGER = "LOGGER";
    private static final String MESSAGE = "MESSAGE";
    private static final String TIMESTAMP = "TIMESTAMP";
    private static final String NDC = "NDC";
    private static final String LEVEL = "LEVEL";
    private static final String THREAD = "THREAD";
    private static final String CLASS = "CLASS";
    private static final String FILE = "FILE";
    private static final String LINE = "LINE";
    private static final String METHOD = "METHOD";
    private static final String NEWLINE = "(LF)";

    private static final String DEFAULT_HOST = "file";

    // all lines other than first line of exception begin with tab followed by 'at' followed by text
    private static final String EXCEPTION_PATTERN = "^\\s+at.*";
    private static final String REGEXP_DEFAULT_WILDCARD = ".*?";
    private static final String REGEXP_GREEDY_WILDCARD = ".*";
    private static final String PATTERN_WILDCARD = "*";
    // pull in optional leading and trailing spaces
    private static final String NOSPACE_GROUP = "(\\s*?\\S*?\\s*?)";
    private static final String DEFAULT_GROUP = "(" + REGEXP_DEFAULT_WILDCARD + ")";
    private static final String GREEDY_GROUP = "(" + REGEXP_GREEDY_WILDCARD + ")";
    private static final String MULTIPLE_SPACES_REGEXP = "[ ]+";
    private static final String NEWLINE_REGEXP = "\n";
    private final String newLine = System.getProperty("line.separator");

    private final String[] emptyException = new String[] {
        ""
    };

    private SimpleDateFormat dateFormat;
    private String timestampFormat = "yyyy-MM-d HH:mm:ss,SSS";
    private String logFormat;
    private String customLevelDefinitions;
    private String filterExpression;

    private static final String VALID_DATEFORMAT_CHARS = "GyMwWDdFEaHkKhmsSzZ";
    private static final String VALID_DATEFORMAT_CHAR_PATTERN = "[" + VALID_DATEFORMAT_CHARS + "]";

    //private Rule expressionRule;

    private Map currentMap;
    private List additionalLines;
    private List matchingKeywords;

    private String regexp;
    //private Reader reader;
    private Pattern regexpPattern;
    private Pattern exceptionPattern;
    private String timestampPatternText;

    private boolean appendNonMatches;
    private final Map customLevelDefinitionMap = new HashMap();

    // default to one line - this number is incremented for each (LF) found in the logFormat
    private int lineCount = 1;

    public LogFilePatternReceiver() {
        keywords.add(TIMESTAMP);
        keywords.add(LOGGER);
        keywords.add(LEVEL);
        keywords.add(THREAD);
        keywords.add(CLASS);
        keywords.add(FILE);
        keywords.add(LINE);
        keywords.add(METHOD);
        keywords.add(MESSAGE);
        keywords.add(NDC);
        try {
            exceptionPattern = Pattern.compile(EXCEPTION_PATTERN);
        } catch (PatternSyntaxException pse) {
            // shouldn't happen
        }
    }

    /**
     * If the log file contains non-log4j level strings, they can be mapped to log4j levels using the format
     * (android example): V=TRACE,D=DEBUG,I=INFO,W=WARN,E=ERROR,F=FATAL,S=OFF
     * 
     * @param customLevelDefinitions the level definition string
     */
    public void setCustomLevelDefinitions(String customLevelDefinitions) {
        this.customLevelDefinitions = customLevelDefinitions;
    }

    public String getCustomLevelDefinitions() {
        return customLevelDefinitions;
    }

    /**
     * Accessor
     * 
     * @return append non matches
     */
    public boolean isAppendNonMatches() {
        return appendNonMatches;
    }

    /**
     * Mutator
     * 
     * @param appendNonMatches
     */
    public void setAppendNonMatches(boolean appendNonMatches) {
        this.appendNonMatches = appendNonMatches;
    }

    /**
     * Accessor
     * 
     * @return filter expression
     */
    public String getFilterExpression() {
        return filterExpression;
    }

    /**
     * Mutator
     * 
     * @param filterExpression
     */
    public void setFilterExpression(String filterExpression) {
        this.filterExpression = filterExpression;
    }

    /**
     * Accessor
     * 
     * @return log format
     */
    public String getLogFormat() {
        return logFormat;
    }

    /**
     * Mutator
     * 
     * @param logFormat the format
     */
    public void setLogFormat(String logFormat) {
        this.logFormat = logFormat;
    }

    /**
     * Mutator. Specify a pattern from {@link java.text.SimpleDateFormat}
     * 
     * @param timestampFormat
     */
    public void setTimestampFormat(String timestampFormat) {
        this.timestampFormat = timestampFormat;
    }

    /**
     * Accessor
     * 
     * @return timestamp format
     */
    public String getTimestampFormat() {
        return timestampFormat;
    }

    /**
     * Walk the additionalLines list, looking for the EXCEPTION_PATTERN.
     * <p>
     * Return the index of the first matched line (the match may be the 1st line of an exception)
     * <p>
     * Assumptions: <br>
     * - the additionalLines list may contain both message and exception lines<br>
     * - message lines are added to the additionalLines list and then exception lines (all message lines occur
     * in the list prior to all exception lines)
     * 
     * @return -1 if no exception line exists, line number otherwise
     */
    private int getExceptionLine() {
        for (int i = 0; i < additionalLines.size(); i++) {
            Matcher exceptionMatcher = exceptionPattern.matcher((String)additionalLines.get(i));
            if (exceptionMatcher.matches()) {
                return i;
            }
        }
        return -1;
    }

    /**
     * Combine all message lines occuring in the additionalLines list, adding a newline character between each
     * line
     * <p>
     * the event will already have a message - combine this message with the message lines in the
     * additionalLines list (all entries prior to the exceptionLine index)
     * 
     * @param firstMessageLine primary message line
     * @param exceptionLine index of first exception line
     * @return message
     */
    private String buildMessage(String firstMessageLine, int exceptionLine) {
        if (additionalLines.size() == 0) {
            return firstMessageLine;
        }
        StringBuffer message = new StringBuffer();
        if (firstMessageLine != null) {
            message.append(firstMessageLine);
        }

        int linesToProcess = (exceptionLine == -1 ? additionalLines.size() : exceptionLine);

        for (int i = 0; i < linesToProcess; i++) {
            message.append(newLine);
            message.append(additionalLines.get(i));
        }
        return message.toString();
    }

    /**
     * Combine all exception lines occuring in the additionalLines list into a String array
     * <p>
     * (all entries equal to or greater than the exceptionLine index)
     * 
     * @param exceptionLine index of first exception line
     * @return exception
     */
    private String[] buildException(int exceptionLine) {
        if (exceptionLine == -1) {
            return emptyException;
        }
        String[] exception = new String[additionalLines.size() - exceptionLine - 1];
        for (int i = 0; i < exception.length; i++) {
            exception[i] = (String)additionalLines.get(i + exceptionLine);
        }
        return exception;
    }

    /**
     * Construct a logging event from currentMap and additionalLines (additionalLines contains multiple
     * message lines and any exception lines)
     * <p>
     * CurrentMap and additionalLines are cleared in the process
     * 
     * @return event
     */
    private LoggingEvent buildEvent() {
        if (currentMap.size() == 0) {
            if (additionalLines.size() > 0) {
                for (Iterator iter = additionalLines.iterator(); iter.hasNext();) {
                    getLogger().info("found non-matching line: " + iter.next());
                }
            }
            additionalLines.clear();
            return null;
        }
        // the current map contains fields - build an event
        int exceptionLine = getExceptionLine();
        String[] exception = buildException(exceptionLine);

        // messages are listed before exceptions in additionallines
        if (additionalLines.size() > 0 && exception.length > 0) {
            currentMap.put(MESSAGE, buildMessage((String)currentMap.get(MESSAGE), exceptionLine));
        }
        LoggingEvent event = convertToEvent(currentMap, exception);
        currentMap.clear();
        additionalLines.clear();
        return event;
    }

    /**
     * Read, parse and optionally tail the log file, converting entries into logging events. A
     * runtimeException is thrown if the logFormat pattern is malformed.
     * 
     * @param bufferedReader
     * @throws IOException
     */
    protected void process(BufferedReader bufferedReader) throws IOException {
        Matcher eventMatcher;
        Matcher exceptionMatcher;
        String line;
        // if newlines are provided in the logFormat - (LF) - combine the lines prior to matching
        while ((line = bufferedReader.readLine()) != null) {
            // there is already one line (read above, start i at 1
            for (int i = 1; i < lineCount; i++) {
                String thisLine = bufferedReader.readLine();
                if (thisLine != null) {
                    line = line + newLine + thisLine;
                }
            }
            eventMatcher = regexpPattern.matcher(line);
            // skip empty line entries
            if (line.trim().equals("")) {
                continue;
            }
            exceptionMatcher = exceptionPattern.matcher(line);
            if (eventMatcher.matches()) {
                // build an event from the previous match (held in current map)
                LoggingEvent event = buildEvent();
                if (event != null) {
                    if (passesExpression(event)) {
                        doPost(event);
                    }
                }
                currentMap.putAll(processEvent(eventMatcher.toMatchResult()));
            } else if (exceptionMatcher.matches()) {
                // an exception line
                additionalLines.add(line);
            } else {
                // neither...either post an event with the line or append as additional lines
                // if this was a logging event with multiple lines, each line will show up as its own event
                // instead of being
                // appended as multiple lines on the same event..
                // choice is to have each non-matching line show up as its own line, or append them all to a
                // previous event
                if (appendNonMatches) {
                    // hold on to the previous time, so we can do our best to preserve time-based ordering if
                    // the event is a non-match
                    String lastTime = (String)currentMap.get(TIMESTAMP);
                    // build an event from the previous match (held in current map)
                    if (currentMap.size() > 0) {
                        LoggingEvent event = buildEvent();
                        if (event != null) {
                            if (passesExpression(event)) {
                                doPost(event);
                            }
                        }
                    }
                    if (lastTime != null) {
                        currentMap.put(TIMESTAMP, lastTime);
                    }
                    currentMap.put(MESSAGE, line);
                } else {
                    additionalLines.add(line);
                }
            }
        }

        // process last event if one exists
        LoggingEvent event = buildEvent();
        if (event != null) {
            if (passesExpression(event)) {
                doPost(event);
            }
        }
    }
    
    public abstract void doPost(final LoggingEvent event);
    
    protected void createPattern() {
        regexpPattern = Pattern.compile(regexp);
    }

    /**
     * Helper method that supports the evaluation of the expression
     * 
     * @param event
     * @return true if expression isn't set, or the result of the evaluation otherwise
     */
    private boolean passesExpression(LoggingEvent event) {
//        if (event != null) {
//            if (expressionRule != null) {
//                return (expressionRule.evaluate(event, null));
//            }
//        }
        return true;
    }

    /**
     * Convert the match into a map.
     * <p>
     * Relies on the fact that the matchingKeywords list is in the same order as the groups in the regular
     * expression
     * 
     * @param result
     * @return map
     */
    private Map processEvent(MatchResult result) {
        Map map = new HashMap();
        // group zero is the entire match - process all other groups
        for (int i = 1; i < result.groupCount() + 1; i++) {
            Object key = matchingKeywords.get(i - 1);
            Object value = result.group(i);
            map.put(key, value);

        }
        return map;
    }

    /**
     * Helper method that will convert timestamp format to a pattern
     * 
     * @return string
     */
    private String convertTimestamp() {
        // some locales (for example, French) generate timestamp text with characters not included in \w -
        // now using \S (all non-whitespace characters) instead of /w
        String result = timestampFormat.replaceAll(VALID_DATEFORMAT_CHAR_PATTERN + "+", "\\\\S+");
        // make sure dots in timestamp are escaped
        result = result.replaceAll(Pattern.quote("."), "\\\\.");
        return result;
    }

    /**
     * Build the regular expression needed to parse log entries
     */
    protected void initialize() {
        currentMap = new HashMap();
        additionalLines = new ArrayList();
        matchingKeywords = new ArrayList();

        if (timestampFormat != null) {
            dateFormat = new SimpleDateFormat(quoteTimeStampChars(timestampFormat));
            timestampPatternText = convertTimestamp();
        }
        // if custom level definitions exist, parse them
        updateCustomLevelDefinitionMap();
//        try {
//            if (filterExpression != null) {
//                expressionRule = ExpressionRule.getRule(filterExpression);
//            }
//        } catch (Exception e) {
//            getLogger().warn("Invalid filter expression: " + filterExpression, e);
//        }

        List buildingKeywords = new ArrayList();

        String newPattern = logFormat;

        // process line feeds - (LF) - in the logFormat - before processing properties
        int index = 0;
        while (index > -1) {
            index = newPattern.indexOf(NEWLINE);
            if (index > -1) {
                // keep track of number of expected newlines in the format, so the lines can be concatenated
                // prior to matching
                lineCount++;
                newPattern = singleReplace(newPattern, NEWLINE, NEWLINE_REGEXP);
            }
        }

        String current = newPattern;
        // build a list of property names and temporarily replace the property with an empty string,
        // we'll rebuild the pattern later
        List propertyNames = new ArrayList();
        index = 0;
        while (index > -1) {
            if (current.indexOf(PROP_START) > -1 && current.indexOf(PROP_END) > -1) {
                index = current.indexOf(PROP_START);
                String longPropertyName = current.substring(current.indexOf(PROP_START),
                                                            current.indexOf(PROP_END) + 1);
                String shortProp = getShortPropertyName(longPropertyName);
                buildingKeywords.add(shortProp);
                propertyNames.add(longPropertyName);
                current = current.substring(longPropertyName.length() + 1 + index);
                newPattern = singleReplace(newPattern, longPropertyName,
                                           new Integer(buildingKeywords.size() - 1).toString());
            } else {
                // no properties
                index = -1;
            }
        }

        /*
         * we're using a treemap, so the index will be used as the key to ensure keywords are ordered
         * correctly examine pattern, adding keywords to an index-based map patterns can contain only one of
         * these per entry...properties are the only 'keyword' that can occur multiple times in an entry
         */
        Iterator iter = keywords.iterator();
        while (iter.hasNext()) {
            String keyword = (String)iter.next();
            int index2 = newPattern.indexOf(keyword);
            if (index2 > -1) {
                buildingKeywords.add(keyword);
                newPattern = singleReplace(newPattern, keyword,
                                           new Integer(buildingKeywords.size() - 1).toString());
            }
        }

        String buildingInt = "";

        for (int i = 0; i < newPattern.length(); i++) {
            String thisValue = String.valueOf(newPattern.substring(i, i + 1));
            if (isInteger(thisValue)) {
                buildingInt = buildingInt + thisValue;
            } else {
                if (isInteger(buildingInt)) {
                    matchingKeywords.add(buildingKeywords.get(Integer.parseInt(buildingInt)));
                }
                // reset
                buildingInt = "";
            }
        }

        // if the very last value is an int, make sure to add it
        if (isInteger(buildingInt)) {
            matchingKeywords.add(buildingKeywords.get(Integer.parseInt(buildingInt)));
        }

        newPattern = replaceMetaChars(newPattern);

        // compress one or more spaces in the pattern into the [ ]+ regexp
        // (supports padding of level in log files)
        newPattern = newPattern.replaceAll(MULTIPLE_SPACES_REGEXP, MULTIPLE_SPACES_REGEXP);
        newPattern = newPattern.replaceAll(Pattern.quote(PATTERN_WILDCARD), REGEXP_DEFAULT_WILDCARD);
        // use buildingKeywords here to ensure correct order
        for (int i = 0; i < buildingKeywords.size(); i++) {
            String keyword = (String)buildingKeywords.get(i);
            // make the final keyword greedy (we're assuming it's the message)
            if (i == (buildingKeywords.size() - 1)) {
                newPattern = singleReplace(newPattern, String.valueOf(i), GREEDY_GROUP);
            } else if (TIMESTAMP.equals(keyword)) {
                newPattern = singleReplace(newPattern, String.valueOf(i), "(" + timestampPatternText + ")");
            } else if (LOGGER.equals(keyword) || LEVEL.equals(keyword)) {
                newPattern = singleReplace(newPattern, String.valueOf(i), NOSPACE_GROUP);
            } else {
                newPattern = singleReplace(newPattern, String.valueOf(i), DEFAULT_GROUP);
            }
        }

        regexp = newPattern;
        getLogger().debug("regexp is " + regexp);
    }

    private void updateCustomLevelDefinitionMap() {
        if (customLevelDefinitions != null) {
            StringTokenizer entryTokenizer = new StringTokenizer(customLevelDefinitions, ",");

            customLevelDefinitionMap.clear();
            while (entryTokenizer.hasMoreTokens()) {
                StringTokenizer innerTokenizer = new StringTokenizer(entryTokenizer.nextToken(), "=");
                customLevelDefinitionMap.put(innerTokenizer.nextToken(),
                                             Level.toLevel(innerTokenizer.nextToken()));
            }
        }
    }

    private boolean isInteger(String value) {
        try {
            Integer.parseInt(value);
            return true;
        } catch (NumberFormatException nfe) {
            return false;
        }
    }

    private String quoteTimeStampChars(String input) {
        // put single quotes around text that isn't a supported dateformat char
        StringBuffer result = new StringBuffer();
        // ok to default to false because we also check for index zero below
        boolean lastCharIsDateFormat = false;
        for (int i = 0; i < input.length(); i++) {
            String thisVal = input.substring(i, i + 1);
            boolean thisCharIsDateFormat = VALID_DATEFORMAT_CHARS.contains(thisVal);
            // we have encountered a non-dateformat char
            if (!thisCharIsDateFormat && (i == 0 || lastCharIsDateFormat)) {
                result.append("'");
            }
            // we have encountered a dateformat char after previously encountering a non-dateformat char
            if (thisCharIsDateFormat && i > 0 && !lastCharIsDateFormat) {
                result.append("'");
            }
            lastCharIsDateFormat = thisCharIsDateFormat;
            result.append(thisVal);
        }
        // append an end single-quote if we ended with non-dateformat char
        if (!lastCharIsDateFormat) {
            result.append("'");
        }
        return result.toString();
    }

    private String singleReplace(String inputString, String oldString, String newString) {
        int propLength = oldString.length();
        int startPos = inputString.indexOf(oldString);
        if (startPos == -1) {
            getLogger().info("string: " + oldString + " not found in input: " + inputString
                                 + " - returning input");
            return inputString;
        }
        if (startPos == 0) {
            inputString = inputString.substring(propLength);
            inputString = newString + inputString;
        } else {
            inputString = inputString.substring(0, startPos) + newString
                          + inputString.substring(startPos + propLength);
        }
        return inputString;
    }

    private String getShortPropertyName(String longPropertyName) {
        String currentProp = longPropertyName.substring(longPropertyName.indexOf(PROP_START));
        String prop = currentProp.substring(0, currentProp.indexOf(PROP_END) + 1);
        String shortProp = prop.substring(PROP_START.length(), prop.length() - 1);
        return shortProp;
    }

    /**
     * Some perl5 characters may occur in the log file format. Escape these characters to prevent parsing
     * errors.
     * 
     * @param input
     * @return string
     */
    private String replaceMetaChars(String input) {
        // escape backslash first since that character is used to escape the remaining meta chars
        input = input.replaceAll("\\\\", "\\\\\\");

        // don't escape star - it's used as the wildcard
        input = input.replaceAll(Pattern.quote("]"), "\\\\]");
        input = input.replaceAll(Pattern.quote("["), "\\\\[");
        input = input.replaceAll(Pattern.quote("^"), "\\\\^");
        input = input.replaceAll(Pattern.quote("$"), "\\\\$");
        input = input.replaceAll(Pattern.quote("."), "\\\\.");
        input = input.replaceAll(Pattern.quote("|"), "\\\\|");
        input = input.replaceAll(Pattern.quote("?"), "\\\\?");
        input = input.replaceAll(Pattern.quote("+"), "\\\\+");
        input = input.replaceAll(Pattern.quote("("), "\\\\(");
        input = input.replaceAll(Pattern.quote(")"), "\\\\)");
        input = input.replaceAll(Pattern.quote("-"), "\\\\-");
        input = input.replaceAll(Pattern.quote("{"), "\\\\{");
        input = input.replaceAll(Pattern.quote("}"), "\\\\}");
        input = input.replaceAll(Pattern.quote("#"), "\\\\#");
        return input;
    }

    /**
     * Convert a keyword-to-values map to a LoggingEvent
     * 
     * @param fieldMap
     * @param exception
     * @return logging event
     */
    private LoggingEvent convertToEvent(Map fieldMap, String[] exception) {
        if (fieldMap == null) {
            return null;
        }

        // a logger must exist at a minimum for the event to be processed
        if (!fieldMap.containsKey(LOGGER)) {
            fieldMap.put(LOGGER, "Unknown");
        }
        if (exception == null) {
            exception = emptyException;
        }

        Logger logger = null;
        long timeStamp = 0L;
        String level = null;
        String threadName = null;
        Object message = null;
        String ndc = null;
        String className = null;
        String methodName = null;
        String eventFileName = null;
        String lineNumber = null;
        Hashtable properties = new Hashtable();

        logger = Logger.getLogger((String)fieldMap.remove(LOGGER));

        if ((dateFormat != null) && fieldMap.containsKey(TIMESTAMP)) {
            try {
                timeStamp = dateFormat.parse((String)fieldMap.remove(TIMESTAMP)).getTime();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        // use current time if timestamp not parseable
        if (timeStamp == 0L) {
            timeStamp = System.currentTimeMillis();
        }

        message = fieldMap.remove(MESSAGE);
        if (message == null) {
            message = "";
        }

        level = (String)fieldMap.remove(LEVEL);
        Level levelImpl;
        if (level == null) {
            levelImpl = Level.DEBUG;
        } else {
            // first try to resolve against custom level definition map, then fall back to regular levels
            levelImpl = (Level)customLevelDefinitionMap.get(level);
            if (levelImpl == null) {
                levelImpl = Level.toLevel(level.trim());
                if (!level.equals(levelImpl.toString())) {
                    // check custom level map
                    if (levelImpl == null) {
                        levelImpl = Level.DEBUG;
                        getLogger().debug("found unexpected level: " + level + ", logger: "
                                              + logger.getName() + ", msg: " + message);
                        // make sure the text that couldn't match a level is added to the message
                        message = level + " " + message;
                    }
                }
            }
        }

        threadName = (String)fieldMap.remove(THREAD);

        ndc = (String)fieldMap.remove(NDC);

        className = (String)fieldMap.remove(CLASS);

        methodName = (String)fieldMap.remove(METHOD);

        eventFileName = (String)fieldMap.remove(FILE);

        lineNumber = (String)fieldMap.remove(LINE);


        // all remaining entries in fieldmap are properties
        properties.putAll(fieldMap);

        LocationInfo info = null;

        if ((eventFileName != null) || (className != null) || (methodName != null) || (lineNumber != null)) {
            info = new LocationInfo(eventFileName, className, methodName, lineNumber);
        } else {
            info = LocationInfo.NA_LOCATION_INFO;
        }

        LoggingEvent event = new LoggingEvent(null, logger, timeStamp, levelImpl, message, threadName,
                                              new ThrowableInformation(exception), ndc, info, properties);

        return event;
    }
    
    protected org.slf4j.Logger getLogger() {
        return logger;
    }
}


package com.infy.gs.automation.log.parser;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.PipedInputStream;
import java.io.PipedOutputStream;
import java.io.Reader;

import org.apache.commons.io.IOUtils;
import org.apache.commons.io.input.Tailer;
import org.apache.commons.io.input.TailerListenerAdapter;
import org.apache.commons.lang.Validate;
import org.apache.log4j.spi.LoggingEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogFileTailer {
    private Logger logger = LoggerFactory.getLogger(getClass());
    private final long pollingMs;
    private final LogFileParser receiver;
    private Tailer tailer;
    private PipedInputStream inputStream;
    private PipedOutputStream outputStream;
    
    public LogFileTailer(final AppenderConfig appenderConfig, 
                         final long pollingMs,
                         final LogDataHandler collector) {
        
        Validate.notNull(appenderConfig);
        Validate.isTrue(pollingMs > 0);
        
        receiver = new LogFileParser(appenderConfig, collector);
        this.pollingMs = pollingMs;
    }
    
    public void doTail(final File logFile) throws IOException {
        final InputStream inputStream = setupTailer(logFile);
        
        Thread thread = new Thread() {
            public void run() {
                try {
                    receiver.process(inputStream);
                } catch(IOException e) {
                    logger.trace("", e);
                }
            }
        };
        thread.setDaemon(true);
        thread.start();
    }
    
    public void stop() {
        tailer.stop();
        IOUtils.closeQuietly(inputStream);
        IOUtils.closeQuietly(outputStream);
    }
    
    private InputStream setupTailer(File logFile) throws IOException {
        inputStream = new PipedInputStream();
        outputStream = new PipedOutputStream(inputStream);
        
        TailerListenerAdapter listener = new TailerListenerAdapter() {
            public void handle(String line) {
                try {
                    outputStream.write((line + IOUtils.LINE_SEPARATOR).getBytes("UTF-8"));
                } catch (IOException e) {
                    logger.trace("", e);
                }
            }
        };

        tailer = new Tailer(logFile, listener, pollingMs, false, true);
        Thread thread = new Thread(tailer);
        thread.setDaemon(true);
        thread.start();
        
        return inputStream;
    }
}


package com.infy.gs.automation.log.parser;

import java.io.File;
import java.net.MalformedURLException;
import java.net.URL;

import org.apache.log4j.LogManager;
import org.apache.log4j.helpers.Loader;
import org.apache.log4j.helpers.OptionConverter;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public final class LoggerUtils {
    private static final Logger LOGGER = LoggerFactory.getLogger(LoggerUtils.class);
    
    private LoggerUtils() {
    }

    /**
     * 
     * 
     * @return
     */
    @SuppressWarnings("deprecation")
    public static File getLogFileProperties() {
        String configurationOptionStr = OptionConverter.getSystemProperty(
               LogManager.DEFAULT_CONFIGURATION_KEY, null);

        URL url = null;
        if (configurationOptionStr != null) {
            try {
                url = new URL(configurationOptionStr);
            } catch (MalformedURLException ex) {
                // so, resource is not a URL:
                // attempt to get the resource from the class path
                url = Loader.getResource(configurationOptionStr);
            }
        } else {
            url = Loader.getResource(LogManager.DEFAULT_CONFIGURATION_FILE);
        }

        if (url != null) {
            File file = new File(url.getFile());
            LOGGER.debug("Log4j Properties {}", file.getAbsolutePath());
            
            return file;
        } else {
            throw new IllegalArgumentException("Cannot load " + LogManager.DEFAULT_CONFIGURATION_FILE);
        }
    }
}

package com.infy.gs.automation.util;

/**
 * @author Tusar
 */
public class KafkaProducerConstants {
	
	public static final long NO_OF_MESSAGES = 3;
	public static final int SLEEP_TIME = 3000;
	public static final String KAFKFA_CLUSTERS = "localhost:9092";
	public static final int KAFKFA_PORT = 9092;
	public static final String INPUT_LOG_LOCATION = "/root/kafka/kafka-producer/loginjectkafka/FACTNYCL2-SEGADEQUACY-DAY-REPORT.log";
	public static final String INPUT_LOG_PROP_LOCATION = "/root/kafka/kafka-producer/loginjectkafka/log4j.properties";
	public static final int KAKFA_PARTITION = 0;
	public static final String KAFKFA_TOPIC = "test";
	public static final long TAIL_CHECK_INTERVAL = 3000;
	public static final int MINUTE_ROUND_INTERVAL = 30; //value should not exceed 30 as there can not be round to hour
}

pom

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>KafkaErrorLogProducer</groupId>
	<artifactId>KafkaErrorLogProducer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<build>
		<sourceDirectory>src</sourceDirectory>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
      			 <plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>exec-maven-plugin</artifactId>
				<version>1.2.1</version>
				<executions>
					<execution>
						<id>gs-poc</id>
						<phase>test</phase>
						<goals>
							<goal>java</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<mainClass>com.infy.gs.automation.log.produce.KafkaLogReader</mainClass>
					<executable>java</executable>
					<arguments>
						<argument>-classpath</argument>
						<argument>target/classes</argument>
						<argument>com.infy.gs.automation.log.produce.KafkaLogReader</argument>
					</arguments>
				</configuration>
			</plugin>  
		</plugins>
	</build>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>
<dependency>
	<groupId>com.google.code.gson</groupId>
	<artifactId>gson</artifactId>
	<version>2.3.1</version>
</dependency>

	
		<dependency>
			<groupId>com.googlecode.json-simple</groupId>
			<artifactId>json-simple</artifactId>
			<version>1.1.1</version>
		</dependency>
		
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-core_2.10</artifactId>
			<version>1.3.1</version>
			<exclusions>
				<exclusion>
					<groupId>org.scala-lang</groupId>
					<artifactId>scala-library</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.10</artifactId>
            <version>1.3.1</version>
        </dependency>
   		<dependency>
      		<groupId>org.apache.spark</groupId>
      		<artifactId>spark-mllib_2.10</artifactId>
      		<version>1.3.1</version>
    	</dependency>		
		
		<!-- MongoDB -->
		
		  <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>casbah-commons_2.10</artifactId>
        <version>2.8.0</version>
    </dependency>
  <dependency>
	<groupId>org.mongodb</groupId>
	<artifactId>mongo-java-driver</artifactId>
	<version>2.13.0</version>
</dependency>

    
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>casbah-query_2.10</artifactId>
        <version>2.8.0</version>
    </dependency>
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>casbah-core_2.10</artifactId>
        <version>2.8.0</version>
    </dependency>
    <dependency>
        <groupId>de.flapdoodle.embed</groupId>
        <artifactId>de.flapdoodle.embed.mongo</artifactId>
        <version>1.46.4</version>
        <scope>test</scope>
    </dependency>
		<dependency>
	<groupId>org.mongodb</groupId>
	<artifactId>mongo-hadoop-core</artifactId>
	<version>1.3.0</version>
</dependency>
		
			
		
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-streaming_2.10</artifactId>
			<version>1.3.1</version>	
			<exclusions>
				<exclusion>
					<groupId>org.scala-lang</groupId>
					<artifactId>scala-library</artifactId>
				</exclusion>
			</exclusions>		
		</dependency>
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-streaming-kafka_2.10</artifactId>
			<version>1.3.1</version>
		</dependency>

		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>net.sf.jopt-simple</groupId>
			<artifactId>jopt-simple</artifactId>
			<version>3.2</version>
		</dependency>
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>14.0.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-log4j12</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
			</exclusions>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-test</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.testng</groupId>
					<artifactId>testng</artifactId>
				</exclusion>
			</exclusions>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka_2.10</artifactId>
			<version>0.8.2.1</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.zookeeper</groupId>
					<artifactId>zookeeper</artifactId>
				</exclusion>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.4</version>
		</dependency>
	<dependency>
		<groupId>org.scala-lang</groupId>
		<artifactId>scala-library</artifactId>
		<version>2.10.4</version>
	</dependency>

		<dependency>
			<groupId>com.msiops.footing</groupId>
			<artifactId>footing-tuple</artifactId>
			<version>0.2</version>
		</dependency>

		<dependency>
			<groupId>commons-cli</groupId>
			<artifactId>commons-cli</artifactId>
			<version>1.2</version>
		</dependency>

		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.4</version>
		</dependency>

	</dependencies>

</project>


log.txt


2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
	at org.hibernate.exception.SQLStateConverter.convert(SQLStateConverter.java:107) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.exception.JDBCExceptionHelper.convert(JDBCExceptionHelper.java:66) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.loader.Loader.loadEntity(Loader.java:2041) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.loader.AbstractEntityLoader.load(AbstractEntityLoader.java:86) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.loader.AbstractEntityLoader.load(AbstractEntityLoader.java:76) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.loader.AbstractEntityLoader.load(AbstractEntityLoader.java:69) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.loader.entity.BatchingEntityLoader.load(BatchingEntityLoader.java:113) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.loader.entity.AbstractEntityPersister.load(AbstractEntityPersister.java:3293) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.event.def.DefaultLoadEventListener.loadFromDataSource(DefaultLoadEventListener.java:496) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.event.def.DefaultLoadEventListener.doLoad(DefaultLoadEventListener.java:477) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.event.def.DefaultLoadEventListener.load(DefaultLoadEventListener.java:277) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.event.def.DefaultLoadEventListener.proxyOrLoad(DefaultLoadEventListener.java:285) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.event.def.DefaultLoadEventListener.onLoad(DefaultLoadEventListener.java:152) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.impl.SessionImpl.fireLoad(SessionImpl.java:1090) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.impl.SessionImpl.get(SessionImpl.java:1005) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.impl.SessionImpl.get(SessionImpl.java:998) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at com.gs.factory.persistence.hibernate.GenericDAOImpl.get(GenericDAOImpl.java:60) ~[persistence-6.7.0.1.jar:na]
	at com.gs.factory.corona.reports.bony.segadequacy.SedAdequacyProcess.findSplPosition(SedAdequacyProcess.java:319) ~[corona-app-4.3.0.5.jar:na]
	at com.gs.factory.corona.reports.bony.segadequacy.SedAdequacyProcess.populateTotalSegAdequate(SedAdequacyProcess.java:283) ~[corona-app-4.3.0.5.jar:na]
	at com.gs.factory.corona.reports.bony.segadequacy.SedAdequacyProcess.groupCandidates(SedAdequacyProcess.java:257) ~[corona-app-4.3.0.5.jar:na]
	at com.gs.factory.corona.reports.bony.segadequacy.SedAdequacyReport.createReport(SedAdequacyProcess.java:49) ~[corona-app-4.3.0.5.jar:na]
	at com.gs.factory.corona.reports.ReportGenerator$1.call(ReportGenerator.java:46) ~[corona-app-4.3.0.5.jar:na]
	at com.gs.factory.corona.reports.ReportGenerator$1.call(ReportGenerator.java:43) ~[corona-app-4.3.0.5.jar:na]
	at com.gs.factory.ste.STEUtils.doTransaction(STEUtils.java:192) ~[ste-6.7.0.1.jar:na]	
	at com.gs.factory.corona.reports.ReportGenerator.process(ReportGenerator.java:43) ~[corona-app-4.3.0.5.jar:na]
	at com.gs.factory.corona.batch.CoronaBatchApplicationBatch.run(CoronaBatchApplicationBatch.java:95) ~[corona-app-4.3.0.5.jar:na]
	at com.gs.factory.common.foundation.GSApplication.start(GSApplication.java:142) ~[common-6.7.0.1.jar:na]
	at com.gs.factory.corona.reports.bony.segadequacy.SedAdequacyReport.main(SedAdequacyProcess.java:37) ~[corona-app-4.3.0.5.jar:na]
Caused by: com.ibm.db2.jcc.b.SQLException: the current traansaction has been rolled back because of deadlock or timeout. Reason code"68".
	at com.ibm.db2.jcc.b.yg.b(yg.java:3046) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.c.eb.h(eb.java:268) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.c.eb.a(eb.java:229) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.c.eb.c(eb.java:33) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.c.u.a(u.java:34) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.c.j.lb(j.java:257) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.b.yg.Q(yg.java:2896) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.c.d.g(d.java:1444) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.b.eb.a(eb.java:191) ~[db2jcc-3.4.65.jar:na]
	at com.ibm.db2.jcc.b.yg.c(yg.java:274) ~[db2jcc-3.4.65.jar:na]	
	at com.ibm.db2.jcc.b.yg.next(yg.java:238) ~[db2jcc-3.4.65.jar:na]
	at org.hibernate.loader.Loader.doQuery(Loader.java:825) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.loader.Loader.doQueryAndInitializeNonLazyCollections(Loader.java:274) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
	at org.hibernate.loader.Loader.loadEntity(Loader.java:2037) ~[hibernate-core-3.6.4.Final.jar:3.6.4.Final]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,013 ERROR [main] com.gs.factory.corona.reports.bony.segadequacy.SegAdequacyReportApplication terminating:caught exception from run method org.hibernate.exception.LockAcquisitionException: could not load an entity: [com.gs.factory.collections.spl.SecuritiesPosition#component[accountId,accountType,businessDate,prodSysynoynm]{prodSynonym=912828K66,businessDate=Friday May 15 00:00:00 EDT 2015,accountId=54513908,accountType=01}]
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-18 07:07:13,017 INFO [Thread-19] org.springframework.context.support.ClassPathXmlApplicationContext Closing CRDDataService ApplicationContext: startup date[Mon May 18 07:01:15 EDT 2015];
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception
2015-05-21 20:23:40,298 WARN[main] com.gs.factory.ste.worker.WorkerGroup worker 'sii-sai-in-1'completed with exception


-------------------------


import java.io.Serializable;
import java.net.UnknownHostException;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

import org.apache.commons.lang3.ArrayUtils;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.mllib.clustering.KMeans;
import org.apache.spark.mllib.clustering.KMeansModel;
import org.apache.spark.mllib.feature.Word2Vec;
import org.apache.spark.mllib.feature.Word2VecModel;
import org.apache.spark.mllib.linalg.DenseVector;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.mllib.regression.LabeledPoint;
import org.apache.spark.mllib.tree.DecisionTree;
import org.apache.spark.mllib.tree.model.DecisionTreeModel;

import com.infy.gs.automation.beans.AlertBean;

import scala.Tuple2;


/**
 * @author Tusar
 * 
 * This class creates a cloud of vectors from words using word2vec.
 * Then using K-Means clustering it identifies the related words and create new categories. 
 * This is useful for auto categorization instead of predefined class.
 * 
 */

class AlertsKMeansClustering implements Serializable {

	
	public static void main(String[] args) throws UnknownHostException {

		SparkConf sparkConf = new SparkConf().setAppName("AlertsDTreeMultiColumns")
				.setMaster("local[2]")
				.set("spark.driver.allowMultipleContexts", "true")
				.set("spark.driver.host", "zeus07")
				.set("spark.driver.port", "8080");
		JavaSparkContext sc = new JavaSparkContext(sparkConf);

		/*
		 * Load Trained CSV file
		 */
		JavaRDD<String> trainFile = sc
				.textFile("/root/kafka/kafka-consumer/decisiontree/sample_autosys.csv");

		// convert to JavaPair with a map between category and alert details

		JavaRDD<AlertBean> rowsTrainRDD = trainFile
				.map(new Function<String, AlertBean>() {
					public AlertBean call(String line) {
						//System.out.println("line######################"+line);
						String[] data = line.split(",");
						//System.out.println("data######################"+data[2]);
						Double points = new Double(0);
						if (data[9] != null && data[9].trim().length() != 0) {
							try {
								points = new Double(data[9].trim());
							} catch (Exception e) {
								//System.out.println("points#######"+data[8].trim()+"#########"+e.getMessage());
								return null;
							}
						}
						
						return new AlertBean(data[0], data[1], data[2],
								data[3], data[4], data[5], data[6], data[7],data[8],
								points);
					}

				});

		// to filter the header from CSV
		JavaRDD<AlertBean> rowsValuesTrainRDD = rowsTrainRDD
				.filter(new Function<AlertBean, Boolean>() {
					@Override
					public Boolean call(AlertBean bean) throws Exception {
						if (bean == null) {
							//System.out.println("Bean False######################");
							return false;
						} else {
							//System.out.println("Bean False######################");
							return true;
							
						}
					}

				});

		// Need list of all words for creating trained model in Word2Vec
		JavaRDD<List<String>> trainRdds = rowsValuesTrainRDD
				.map(new Function<AlertBean, List<String>>() {
					public List<String> call(AlertBean bean) throws Exception {
						// multiple columns to be trained
						String alertDt = clean(bean.getAlertDetail());
						//System.out.println("alertDt######################"+alertDt);
						String appName = clean(bean.getApplicationName());
						//System.out.println(appName);
						String npoints = bean.getNpoints().toString();
						String[] words = ArrayUtils.addAll(
								ArrayUtils.addAll(alertDt.split(" "),
										appName.split(" ")), npoints.split(" "));
						return Arrays.asList(words);

					}

				});

		Word2Vec word2Vec = new Word2Vec();
		Word2VecModel model = word2Vec.fit(trainRdds);
		
		// Need list of all words for creating trained model in Word2Vec
		JavaRDD<Vector> alertVectors = rowsValuesTrainRDD
				.map(new Function<AlertBean, Vector>() {
					public Vector call(AlertBean bean) throws Exception {
						String alertDt = clean(bean.getAlertDetail());
						String appName = clean(bean.getApplicationName());
						String npoints = bean.getNpoints().toString();
						String[] words = ArrayUtils.addAll(
								ArrayUtils.addAll(alertDt.split(" "),
										appName.split(" ")), npoints.split(" "));
						return wordToVector(words, model);

					}

				});
		
		JavaPairRDD<AlertBean, Vector> alertPairVectors = rowsValuesTrainRDD
				.mapToPair(new PairFunction<AlertBean, AlertBean, Vector>() {
					public Tuple2<AlertBean, Vector> call(AlertBean bean) throws Exception {
						String alertDt = clean(bean.getAlertDetail());
						String appName = clean(bean.getApplicationName());
						String npoints = bean.getNpoints().toString();
						String[] words = ArrayUtils.addAll(
								ArrayUtils.addAll(alertDt.split(" "),
										appName.split(" ")), npoints.split(" "));
						return new Tuple2<AlertBean, Vector> (bean, wordToVector(words, model));

					}

				});
		
		int numClusters = 100;
		int numIterations = 25;

		KMeansModel clusters = KMeans.train(alertVectors.rdd(), numClusters, numIterations);
		Double wssse = clusters.computeCost(alertVectors.rdd());

	/*	
		 val article_membership = title_pairs.map(x => (clusters.predict(x._2), x._1))
				    val cluster_centers = sc.parallelize(clusters.clusterCenters.zipWithIndex.map{ e => (e._2,e._1)})
				    val cluster_topics = cluster_centers.mapValues(x => model.findSynonyms(x,5).map(x => x._1))

				    var sample_topic = cluster_topics.take(12)
				    var sample_members = article_membership.filter(x => x._1 == 6).take(10)
				    for (i <- 6 until 12) {
					println("Topic Group #"+i)
					println(sample_topic(i)._2.mkString(","))
					println("-----------------------------")
					sample_members = article_membership.filter(x => x._1 == i).take(10)
					sample_members.foreach{x => println(x._2.mkString(" "))}
					println("-----------------------------")
				    }
		*/

	}

	

	public static double[] sumArray(double[] m, double[] n) {
		for (int i = 0; i < m.length; i++) {
			m[i] = m[i] + n[i];
		}

		return m;
	}

	// This is to calculate the average of vectors
	public static double[] divArray(double[] m, double divisor) {
		for (int i = 0; i < m.length; i++) {
			m[i] = m[i] / divisor;
		}
		return m;
	}

	public static Vector wordToVector(String w, Word2VecModel model) {
		try {
			return model.transform(w);
		} catch (Exception e) {
			return Vectors.zeros(100);
		}
	}

	public static Vector wordToVector(String[] ws, Word2VecModel model) {
		double[] totalSum = wordToVector(ws[0], model).toArray();
		for (int i = 1; i < ws.length; i++) {
			totalSum = sumArray(totalSum, wordToVector(ws[i], model).toArray());

		}
		divArray(totalSum, ws.length);
		return new DenseVector(divArray(totalSum, ws.length));

	}
	
	/* clean the punctuation, stop words and numbers, remove extra space
	 *	and convert to lower case
	 */
	public static String clean(String a){
		if(a != null){
			return a.replaceAll("[^a-zA-Z\\s]", "").replaceAll("\\s+", " ").toLowerCase();
		}else{
			return "";
		}
			
	}

}

-----------------------

import java.io.Serializable;
import java.net.UnknownHostException;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;




import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.mllib.classification.NaiveBayes;
import org.apache.spark.mllib.classification.NaiveBayesModel;
import org.apache.spark.mllib.feature.HashingTF;
import org.apache.spark.mllib.regression.LabeledPoint;
import org.apache.spark.mllib.tree.DecisionTree;
import org.apache.spark.mllib.tree.model.DecisionTreeModel;

import scala.Tuple2;


class AlertDTreeHashTFClassifier implements Serializable {

	/**
	 * Create one Map to assign numeric value to predefined categories.
	 */
	static final Map<String, Double> Category = new HashMap<String, Double>() {
		{
			put("JobFailure", 1.0);
			put("Maxrun", 2.0);
			put("Capacity", 3.0);
			put("ProcessBadMessage", 4.0);
			put("EMS", 5.0);
			put("", 0.0);
		}
	};

	public static void main(String[] args) throws UnknownHostException {

		SparkConf sparkConf = new SparkConf().setAppName("AlertNBClassifier")
				.setMaster("local[2]")
				.set("spark.driver.allowMultipleContexts", "true")
				.set("spark.driver.host", "zeus07")
				.set("spark.driver.port", "8080");
		JavaSparkContext sc = new JavaSparkContext(sparkConf);
		/*
		 * Hashing term frequency vectorizer with 2k features This will generate
		 * unique double value for each word
		 */
		HashingTF htf = new HashingTF(2000);

		/*
		 * Load Trained CSV file 
		 */
		JavaRDD<String> trainFile = sc
				.textFile("/root/kafka/kafka-consumer/classification/sample_autosys.csv");

		

	

		// convert to JavaPair with a map between category and alert details
		@SuppressWarnings("serial")
		JavaPairRDD<String, String> rowsTrainRDD = trainFile
				.mapToPair(new PairFunction<String, String, String>() {
					public Tuple2<String, String> call(String line) {
						String[] data = line.split(",");
						return new Tuple2<String, String>(data[1], data[3]);
					}

				});
		
		Tuple2<String, String> headerTrain = rowsTrainRDD.first();
		
		//to filter the header from CSV
		JavaPairRDD<String, String> rowsValuesTrainRDD = rowsTrainRDD.filter(new Function<Tuple2<String, String>, Boolean>(){
			@Override
			public Boolean call(Tuple2<String, String> tuple) throws Exception {
				//System.out.println(headerTrain._1 +":" + tuple._1);
				if(headerTrain._1 != null && tuple._1 != null && headerTrain._1.equals(tuple._1)){					
					return false;
				}else{
					return true;
				}
			}
			
		});
		
		
		JavaRDD<LabeledPoint> trainPoints = rowsValuesTrainRDD.map(

		new Function<Tuple2<String, String>, LabeledPoint>() {

			// clean the punctuation, stop words and numbers, remove extra space
			// and convert to lower case
			public LabeledPoint call(Tuple2<String, String> tuple)
					throws Exception {
				
					String alertDt = tuple._2.replaceAll("[^a-zA-Z\\s]", "")
						.replaceAll("\\s+", " ").toLowerCase();
					//System.out.println(tuple._1+":"+ Category.get(tuple._1));
					return new LabeledPoint(Category.get(tuple._1), htf
						.transform(Arrays.asList(alertDt.split(" "))));
				
			}

		});

		JavaRDD<String> testFile = sc.textFile("/root/kafka/kafka-consumer/classification/test_data2.csv");		

		// convert to JavaPair
		@SuppressWarnings("serial")
		JavaPairRDD<String, String> rowsTestRDD = testFile
				.mapToPair(new PairFunction<String, String, String>() {
					public Tuple2<String, String> call(String line) {
						String[] data = line.split(",");
						return new Tuple2<String, String>(data[1], data[3]);
					}

				});
		
		Tuple2<String, String> headerTest = rowsTestRDD.first();
		
		//to filter the header from CSV
		JavaPairRDD<String, String> rowsValuesTestRDD = rowsTestRDD.filter(new Function<Tuple2<String, String>, Boolean>(){
			@Override
			public Boolean call(Tuple2<String, String> tuple) throws Exception {
			//System.out.println(headerTest._1 +":" + tuple._1);
				if(headerTest._1 != null && tuple._1 != null && headerTest._1.equals(tuple._1)){
					return false;
				}else{
					return true;
				}
			}
			
		});

		JavaPairRDD<String, LabeledPoint> testPoints = rowsValuesTestRDD
				.mapToPair(new PairFunction<Tuple2<String, String>, String, LabeledPoint>() {
					@Override
					public Tuple2<String, LabeledPoint> call(
							Tuple2<String, String> labelAlerts)
							throws Exception {
						
						String alertDt = labelAlerts._2
								.replaceAll("[^a-zA-Z\\s]", "")
								.replaceAll("\\s+", " ").toLowerCase();
						//System.out.println("alertDt test" + alertDt);
						//System.out.println(" tranform -"+htf.transform(Arrays.asList(alertDt.split(" "))).toArray());
						return new Tuple2<String, LabeledPoint>(labelAlerts._2,
								new LabeledPoint(0.0, htf.transform(Arrays
										.asList(alertDt.split(" ")))));
						
					}

				});
		
		// Train a DecisionTree model for classification.
		Integer numClasses = 10; //this value depends on the Hashmap size for predefined categories
		Map<Integer, Integer> categoricalFeaturesInfo = new HashMap<Integer, Integer>(); //empty value means features are dynamic
		String impurity = "gini";
		Integer maxDepth = 5;
		Integer maxBins = 32; //more value makes to consider more split values for fine-grained split decisions

		final DecisionTreeModel model = DecisionTree.trainClassifier(trainPoints, numClasses,
				  categoricalFeaturesInfo, impurity, maxDepth, maxBins);


		JavaPairRDD<String, Double> predictionAndLabel = testPoints
				.mapToPair(new PairFunction<Tuple2<String, LabeledPoint>, String, Double>() {
					@Override
					public Tuple2<String, Double> call(
							Tuple2<String, LabeledPoint> tuple) {
						//System.out.println("After prediction "+model.predict(tuple._2.features()));
						return new Tuple2<String, Double>(tuple._1, model
								.predict(tuple._2.features()));
					}
				});

		List<Tuple2<String, Double>> alertCategoryList = predictionAndLabel
				.collect();

		for (Tuple2<String, Double> tuple : alertCategoryList) {
			System.out.println("Alert :" + tuple._1 + " , Category :"
					+ getKeyByValue(tuple._2));
		}

	}
	
	public static  String getKeyByValue( Double value) {
	    for (Entry<String, Double> entry : Category.entrySet()) {
	        if (value.doubleValue() == entry.getValue().doubleValue()) {
	            return entry.getKey();
	        }
	    }
	    return null;
	}

}



-----------------


import java.io.Serializable;
import java.net.UnknownHostException;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;


import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.mllib.classification.NaiveBayes;
import org.apache.spark.mllib.classification.NaiveBayesModel;
import org.apache.spark.mllib.feature.HashingTF;
import org.apache.spark.mllib.regression.LabeledPoint;

import scala.Tuple2;


class AlertNBClassifier implements Serializable {

	/**
	 * Create one Map to assign numeric value to predefined categories.
	 */
	static final Map<String, Double> Category = new HashMap<String, Double>() {
		{
			put("JobFailure", 1.0);
			put("Maxrun", 2.0);
			put("Capacity", 3.0);
			put("ProcessBadMessage", 4.0);
			put("EMS", 5.0);
			put("", 0.0);
		}
	};

	public static void main(String[] args) throws UnknownHostException {

		SparkConf sparkConf = new SparkConf().setAppName("AlertNBClassifier")
				.setMaster("local[2]")
				.set("spark.driver.allowMultipleContexts", "true")
				.set("spark.driver.host", "zeus07")
				.set("spark.driver.port", "8080");
		JavaSparkContext sc = new JavaSparkContext(sparkConf);
		/*
		 * Hashing term frequency vectorizer with 2k features This will generate
		 * unique double value for each word
		 */
		HashingTF htf = new HashingTF(2000);

		/*
		 * Load Trained CSV file 
		 */
		JavaRDD<String> trainFile = sc
				.textFile("/root/kafka/kafka-consumer/classification/sample_autosys.csv");

		

	

		// convert to JavaPair with a map between category and alert details
		@SuppressWarnings("serial")
		JavaPairRDD<String, String> rowsTrainRDD = trainFile
				.mapToPair(new PairFunction<String, String, String>() {
					public Tuple2<String, String> call(String line) {
						String[] data = line.split(",");
						return new Tuple2<String, String>(data[1], data[3]);
					}

				});
		
		Tuple2<String, String> headerTrain = rowsTrainRDD.first();
		
		//to filter the header from CSV
		JavaPairRDD<String, String> rowsValuesTrainRDD = rowsTrainRDD.filter(new Function<Tuple2<String, String>, Boolean>(){
			@Override
			public Boolean call(Tuple2<String, String> tuple) throws Exception {
				//System.out.println(headerTrain._1 +":" + tuple._1);
				if(headerTrain._1 != null && tuple._1 != null && headerTrain._1.equals(tuple._1)){					
					return false;
				}else{
					return true;
				}
			}
			
		});
		
		
		JavaRDD<LabeledPoint> trainPoints = rowsValuesTrainRDD.map(

		new Function<Tuple2<String, String>, LabeledPoint>() {

			// clean the punctuation, stop words and numbers, remove extra space
			// and convert to lower case
			public LabeledPoint call(Tuple2<String, String> tuple)
					throws Exception {
				
					String alertDt = tuple._2.replaceAll("[^a-zA-Z\\s]", "")
						.replaceAll("\\s+", " ").toLowerCase();
					//System.out.println(tuple._1+":"+ Category.get(tuple._1));
					return new LabeledPoint(Category.get(tuple._1), htf
						.transform(Arrays.asList(alertDt.split(" "))));
				
			}

		});

		JavaRDD<String> testFile = sc.textFile("/root/kafka/kafka-consumer/classification/test_data2.csv");		

		// convert to JavaPair
		@SuppressWarnings("serial")
		JavaPairRDD<String, String> rowsTestRDD = testFile
				.mapToPair(new PairFunction<String, String, String>() {
					public Tuple2<String, String> call(String line) {
						String[] data = line.split(",");
						return new Tuple2<String, String>(data[1], data[3]);
					}

				});
		
		Tuple2<String, String> headerTest = rowsTestRDD.first();
		
		//to filter the header from CSV
		JavaPairRDD<String, String> rowsValuesTestRDD = rowsTestRDD.filter(new Function<Tuple2<String, String>, Boolean>(){
			@Override
			public Boolean call(Tuple2<String, String> tuple) throws Exception {
			//System.out.println(headerTest._1 +":" + tuple._1);
				if(headerTest._1 != null && tuple._1 != null && headerTest._1.equals(tuple._1)){
					return false;
				}else{
					return true;
				}
			}
			
		});

		JavaPairRDD<String, LabeledPoint> testPoints = rowsValuesTestRDD
				.mapToPair(new PairFunction<Tuple2<String, String>, String, LabeledPoint>() {
					@Override
					public Tuple2<String, LabeledPoint> call(
							Tuple2<String, String> labelAlerts)
							throws Exception {
						
						String alertDt = labelAlerts._2
								.replaceAll("[^a-zA-Z\\s]", "")
								.replaceAll("\\s+", " ").toLowerCase();
						//System.out.println("alertDt test" + alertDt);
						//System.out.println(" tranform -"+htf.transform(Arrays.asList(alertDt.split(" "))).toArray());
						return new Tuple2<String, LabeledPoint>(labelAlerts._2,
								new LabeledPoint(0.0, htf.transform(Arrays
										.asList(alertDt.split(" ")))));
						
					}

				});

		final NaiveBayesModel model = NaiveBayes.train(trainPoints.rdd(), 1.0);

		JavaPairRDD<String, Double> predictionAndLabel = testPoints
				.mapToPair(new PairFunction<Tuple2<String, LabeledPoint>, String, Double>() {
					@Override
					public Tuple2<String, Double> call(
							Tuple2<String, LabeledPoint> tuple) {
						//System.out.println("After prediction "+model.predict(tuple._2.features()));
						return new Tuple2<String, Double>(tuple._1, model
								.predict(tuple._2.features()));
					}
				});

		List<Tuple2<String, Double>> alertCategoryList = predictionAndLabel
				.collect();

		for (Tuple2<String, Double> tuple : alertCategoryList) {
			System.out.println("Alert :" + tuple._1 + " , Category :"
					+ getKeyByValue(tuple._2));
		}

	}
	
	public static  String getKeyByValue( Double value) {
	    for (Entry<String, Double> entry : Category.entrySet()) {
	        if (value.doubleValue() == entry.getValue().doubleValue()) {
	            return entry.getKey();
	        }
	    }
	    return null;
	}

}



-----------------

import java.io.Serializable;
import java.net.UnknownHostException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.mllib.feature.Word2Vec;
import org.apache.spark.mllib.feature.Word2VecModel;
import org.apache.spark.mllib.linalg.DenseVector;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.mllib.regression.LabeledPoint;
import org.apache.spark.mllib.tree.DecisionTree;
import org.apache.spark.mllib.tree.model.DecisionTreeModel;

import scala.Tuple2;


class AlertsDTreeWordToVecClassifier implements Serializable {


	/**
	 * Create one Map to assign numeric value to predefined categories.
	 */
	static final Map<String, Double> Category = new HashMap<String, Double>() {
		{
			put("JobFailure", 1.0);
			put("Maxrun", 2.0);
			put("Capacity", 3.0);
			put("ProcessBadMessage", 4.0);
			put("EMS", 5.0);
			put("", 0.0);
		}
	};

	public static void main(String[] args) throws UnknownHostException {

		SparkConf sparkConf = new SparkConf().setAppName("AlertNBClassifier")
				.setMaster("local[2]")
				.set("spark.driver.allowMultipleContexts", "true")
				.set("spark.driver.host", "zeus07")
				.set("spark.driver.port", "8080");
		JavaSparkContext sc = new JavaSparkContext(sparkConf);
		
		
		

		/*
		 * Load Trained CSV file 
		 */
		JavaRDD<String> trainFile = sc
				.textFile("/root/kafka/kafka-consumer/classification/sample_autosys.csv");

		

	

		// convert to JavaPair with a map between category and alert details
		@SuppressWarnings("serial")
		JavaPairRDD<String, String> rowsTrainRDD = trainFile
				.mapToPair(new PairFunction<String, String, String>() {
					public Tuple2<String, String> call(String line) {
						String[] data = line.split(",");
						return new Tuple2<String, String>(data[1], data[3]);
					}

				});
		
		Tuple2<String, String> headerTrain = rowsTrainRDD.first();
		
		//to filter the header from CSV
		JavaPairRDD<String, String> rowsValuesTrainRDD = rowsTrainRDD.filter(new Function<Tuple2<String, String>, Boolean>(){
			@Override
			public Boolean call(Tuple2<String, String> tuple) throws Exception {
				//System.out.println(headerTrain._1 +":" + tuple._1);
				if(headerTrain._1 != null && tuple._1 != null && headerTrain._1.equals(tuple._1)){					
					return false;
				}else{
					return true;
				}
			}
			
		});
			
		JavaRDD<String> rdd = trainFile
				.map(new Function<String, String>() {
					public  String call(String line) {
						String[] data = line.split(",");
						return data[3];
					}

				});
		JavaRDD<List<String>> trainRdds = rdd.map(
				new Function<String, List<String>>() {
					// clean the punctuation, stop words and numbers, remove extra space
					// and convert to lower case
					public List<String> call(String alert)
							throws Exception {						
							String alertDt = alert.replaceAll("[^a-zA-Z\\s]", "")
								.replaceAll("\\s+", " ").toLowerCase();
							//System.out.println(tuple._1+":"+ Category.get(tuple._1));
							return Arrays.asList(alertDt.split(" "));
							
					}

				});
		
		
		Word2Vec word2Vec = new Word2Vec();
		Word2VecModel model = word2Vec.fit(trainRdds);
		
		JavaRDD<LabeledPoint> trainPoints = rowsValuesTrainRDD.map(
				new Function<Tuple2<String, String>, LabeledPoint>() {

					// clean the punctuation, stop words and numbers, remove extra space
					// and convert to lower case
					public LabeledPoint call(Tuple2<String, String> tuple)
							throws Exception {						
							String alertDt = tuple._2.replaceAll("[^a-zA-Z\\s]", "")
								.replaceAll("\\s+", " ").toLowerCase();
							//System.out.println(tuple._1+":"+ Category.get(tuple._1));
							return new LabeledPoint(Category.get(tuple._1), wordToVector(alertDt.split(" ") ,model));
						
					}

				});
		
		JavaRDD<String> testFile = sc.textFile("/root/kafka/kafka-consumer/classification/test_data2.csv");		

		// convert to JavaPair
		@SuppressWarnings("serial")
		JavaPairRDD<String, String> rowsTestRDD = testFile
				.mapToPair(new PairFunction<String, String, String>() {
					public Tuple2<String, String> call(String line) {
						String[] data = line.split(",");
						return new Tuple2<String, String>(data[1], data[3]);
					}

				});
		
		Tuple2<String, String> headerTest = rowsTestRDD.first();
		
		//to filter the header from CSV
		JavaPairRDD<String, String> rowsValuesTestRDD = rowsTestRDD.filter(new Function<Tuple2<String, String>, Boolean>(){
			@Override
			public Boolean call(Tuple2<String, String> tuple) throws Exception {
			//System.out.println(headerTest._1 +":" + tuple._1);
				if(headerTest._1 != null && tuple._1 != null && headerTest._1.equals(tuple._1)){
					return false;
				}else{
					return true;
				}
			}
			
		});

		JavaPairRDD<String, LabeledPoint> testPoints = rowsValuesTestRDD
				.mapToPair(new PairFunction<Tuple2<String, String>, String, LabeledPoint>() {
					@Override
					public Tuple2<String, LabeledPoint> call(
							Tuple2<String, String> labelAlerts)
							throws Exception {
						
						String alertDt = labelAlerts._2
								.replaceAll("[^a-zA-Z\\s]", "")
								.replaceAll("\\s+", " ").toLowerCase();
						//System.out.println("alertDt test" + alertDt);
						//System.out.println(" tranform -"+htf.transform(Arrays.asList(alertDt.split(" "))).toArray());
											
						return new Tuple2<String, LabeledPoint>(labelAlerts._2,
								new LabeledPoint(0.0,wordToVector(alertDt.split(" ") ,model)));
						
					}

				});
		
		// Train a DecisionTree model for classification.
		Integer numClasses = 10; //this value depends on the Hashmap size for predefined categories
		Map<Integer, Integer> categoricalFeaturesInfo = new HashMap<Integer, Integer>(); //empty value means features are dynamic
		String impurity = "gini";
		Integer maxDepth = 5;
		Integer maxBins = 32; //more value makes to consider more split values for fine-grained split decisions

		final DecisionTreeModel dTreeModel = DecisionTree.trainClassifier(trainPoints, numClasses,
				  categoricalFeaturesInfo, impurity, maxDepth, maxBins);


		JavaPairRDD<String, Double> predictionAndLabel = testPoints
				.mapToPair(new PairFunction<Tuple2<String, LabeledPoint>, String, Double>() {
					@Override
					public Tuple2<String, Double> call(
							Tuple2<String, LabeledPoint> tuple) {
						//System.out.println("After prediction "+model.predict(tuple._2.features()));
						return new Tuple2<String, Double>(tuple._1, dTreeModel
								.predict(tuple._2.features()));
					}
				});

		List<Tuple2<String, Double>> alertCategoryList = predictionAndLabel
				.collect();

		for (Tuple2<String, Double> tuple : alertCategoryList) {
			System.out.println("Alert :" + tuple._1 + " , Category :"
					+ getKeyByValue(tuple._2));
		}

	}
	

	
	public static  String getKeyByValue( Double value) {
	    for (Entry<String, Double> entry : Category.entrySet()) {
	        if (value.doubleValue() == entry.getValue().doubleValue()) {
	            return entry.getKey();
	        }
	    }
	    return null;
	}
	public static double[] sumArray (double[] m, double[] n) {
			for(int i=0; i<m.length; i++){
				m[i] = m[i]+n[i];
			}
		    
		    return m;
	}

	//This is to calculate the average of vectors
		public static double[] divArray (double[] m, double divisor) {
			for(int i=0; i<m.length; i++){
				m[i] = m[i]/divisor;
			}
			return m;
	  }

	public static Vector wordToVector (String w, Word2VecModel model){
		    try {
		      return model.transform(w);
		    } catch (Exception e){
		      return Vectors.zeros(100);
		    }
		  }
	
	public static Vector wordToVector (String[] ws,  Word2VecModel model) {			
			double[] totalSum = wordToVector(ws[0], model).toArray();
			for(int i=1; i<ws.length; i++ ){
				totalSum = sumArray(totalSum , wordToVector(ws[i], model).toArray());				
				
			}
			divArray(totalSum, ws.length);
			return new DenseVector(divArray(totalSum, ws.length));
			
	  }
	
}



-----------------


import java.io.Serializable;
import java.net.UnknownHostException;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

import org.apache.commons.lang3.ArrayUtils;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.mllib.feature.Word2Vec;
import org.apache.spark.mllib.feature.Word2VecModel;
import org.apache.spark.mllib.linalg.DenseVector;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.mllib.regression.LabeledPoint;
import org.apache.spark.mllib.tree.DecisionTree;
import org.apache.spark.mllib.tree.model.DecisionTreeModel;



import com.infy.gs.automation.beans.AlertBean;

import scala.Tuple2;


/**
 * @author Tusar
 * 
 * This class loads the data from two CSV files, one is for train and another is for test.
 * The columns selected for creating the decision model (factors) are Application, Alert Detail (both holds text type) 
 * and NPoints (contains numeric type data).
 * LabelPoint is map between a class/label(category field) and factors (the above three fields) and used for classification.
 * In test file, the value of category field is blank as these values will be predicted by DTree model. 
 * 
 */

class AlertsDTreeWordToVecMultiColumns implements Serializable {

	/**
	 * Create one Map to assign numeric value to predefined categories.
	 */
	static final Map<String, Double> Category = new HashMap<String, Double>() {
		{
			put("JobFailure", 1.0);
			put("Maxrun", 2.0);
			put("Capacity", 3.0);
			put("ProcessBadMessage", 4.0);
			put("EMS", 5.0);
			put("", 0.0);
		}
	};

	public static void main(String[] args) throws UnknownHostException {

		SparkConf sparkConf = new SparkConf().setAppName("AlertsDTreeMultiColumns")
				.setMaster("local[2]")
				.set("spark.driver.allowMultipleContexts", "true");
				//.set("spark.driver.host", "zeus07")
				//.set("spark.driver.port", "8080
		JavaSparkContext sc = new JavaSparkContext(sparkConf);

		/*
		 * Load Trained CSV file
		 */
		JavaRDD<String> trainFile = sc
				.textFile("sample_autosys.csv");

		// convert to JavaPair with a map between category and alert details

		JavaRDD<AlertBean> rowsTrainRDD = trainFile
				.map(new Function<String, AlertBean>() {
					public AlertBean call(String line) {
						//System.out.println("line######################"+line);
						String[] data = line.split(",");
						//System.out.println("data######################"+data[2]);
						Double points = new Double(0);
						if (data[9] != null && data[9].trim().length() != 0) {
							try {
								points = new Double(data[9].trim());
							} catch (Exception e) {
								//System.out.println("points#######"+data[8].trim()+"#########"+e.getMessage());
								return null;
							}
						}
						
						return new AlertBean(data[0], data[1], data[2],
								data[3], data[4], data[5], data[6], data[7],data[8],
								points);
					}

				});

		// to filter the header from CSV
		JavaRDD<AlertBean> rowsValuesTrainRDD = rowsTrainRDD
				.filter(new Function<AlertBean, Boolean>() {
					public Boolean call(AlertBean bean) throws Exception {
						if (bean == null) {
							//System.out.println("Bean False######################");
							return false;
						} else {
							//System.out.println("Bean False######################");
							return true;
							
						}
					}

				});

		//check bean != null
		// Need list of all words for creating trained model in Word2Vec
		JavaRDD<List<String>> trainRdds = rowsValuesTrainRDD
				.map(new Function<AlertBean, List<String>>() {
					public List<String> call(AlertBean bean) throws Exception {
						// multiple columns to be trained
						String alertDt = clean(bean.getAlertDetail());
						//System.out.println("alertDt######################"+alertDt);
						String appName = clean(bean.getApplicationName());
						//System.out.println(appName);
						String npoints = bean.getNpoints().toString();
						String[] words = ArrayUtils.addAll(
								ArrayUtils.addAll(alertDt.split(" "),
										appName.split(" ")), npoints.split(" "));
						return Arrays.asList(words);

					}

				});

		Word2Vec word2Vec = new Word2Vec();
		Word2VecModel model = word2Vec.fit(trainRdds);

		JavaRDD<LabeledPoint> trainPoints = rowsValuesTrainRDD
				.map(new Function<AlertBean, LabeledPoint>() {

					
					public LabeledPoint call(AlertBean bean) throws Exception {
						String alertDt = clean(bean.getAlertDetail());
						String appName = clean(bean.getApplicationName());
						String npoints = bean.getNpoints().toString();
						String[] words = ArrayUtils.addAll(
								ArrayUtils.addAll(alertDt.split(" "),
										appName.split(" ")), npoints.split(" "));
						return new LabeledPoint(
								Category.get(bean.getCategory()), wordToVector(
										words, model));

					}

				});

		JavaRDD<String> testFile = sc
				.textFile("/root/kafka/kafka-consumer/decisiontree/test_data2.csv");

		@SuppressWarnings("serial")
		JavaRDD<AlertBean> rowsTestRDD = testFile
				.map(new Function<String, AlertBean>() {
					public AlertBean call(String line) {
						String[] data = line.split(",");
						Double points = new Double(0);
						if (data[9] != null && data[9].trim().length() != 0) {
							try {
								points = new Double(data[9].trim());
							} catch (Exception e) {
								return null;
							}
						}
						return new AlertBean(data[0], data[1], data[2],
								data[3], data[4], data[5], data[6], data[7],data[8],
								points);
					}

				});

		// to filter the header from CSV
		JavaRDD<AlertBean> rowsValuesTestRDD = rowsTestRDD
				.filter(new Function<AlertBean, Boolean>() {
					@Override
					public Boolean call(AlertBean bean) throws Exception {
						if (bean == null) {
							return false;
						} else {
							return true;
						}
					}

				});

		JavaPairRDD<String, LabeledPoint> testPoints = rowsValuesTestRDD
				.mapToPair(new PairFunction<AlertBean, String, LabeledPoint>() {
					@Override
					public Tuple2<String, LabeledPoint> call(AlertBean bean)
							throws Exception {
						// considering multiple columns for model
						String alertDt = clean(bean.getAlertDetail());
						String appName = clean(bean.getApplicationName());
						String npoints = bean.getNpoints().toString();
						String[] words = ArrayUtils.addAll(
								ArrayUtils.addAll(alertDt.split(" "),
										appName.split(" ")), npoints.split(" "));

						return new Tuple2<String, LabeledPoint>(bean.getAlertDetail(),
								new LabeledPoint(0.0,
										wordToVector(words, model)));

					}

				});

		
		
		/*
		 * this value depends on the Hashmap size for predefined categories
		 */
		Integer numClasses = 10; 
		/* empty value means features are dynamic*/
		Map<Integer, Integer> categoricalFeaturesInfo = new HashMap<Integer, Integer>(); 
		String impurity = "gini";
		Integer maxDepth = 7;
		/*
		 * more value makes to consider more split valuesfor fine-grained split decisions
		 */
		Integer maxBins = 50; 
		
		// Train a DecisionTree model for classification.
		final DecisionTreeModel dTreeModel = DecisionTree.trainClassifier(
				trainPoints, numClasses, categoricalFeaturesInfo, impurity,
				maxDepth, maxBins);

		JavaPairRDD<String, Double> predictionAndLabel = testPoints
				.mapToPair(new PairFunction<Tuple2<String, LabeledPoint>, String, Double>() {
					@Override
					public Tuple2<String, Double> call(
							Tuple2<String, LabeledPoint> tuple) {
						// System.out.println("After prediction "+model.predict(tuple._2.features()));
						return new Tuple2<String, Double>(tuple._1, dTreeModel
								.predict(tuple._2.features()));
					}
				});

		List<Tuple2<String, Double>> alertCategoryList = predictionAndLabel
				.collect();

		for (Tuple2<String, Double> tuple : alertCategoryList) {
			System.out.println("Alert :" + tuple._1 + " , Category :"
					+ getKeyByValue(tuple._2));
		}

	}

	public static String getKeyByValue(Double value) {
		for (Entry<String, Double> entry : Category.entrySet()) {
			if (value.doubleValue() == entry.getValue().doubleValue()) {
				return entry.getKey();
			}
		}
		return null;
	}

	public static double[] sumArray(double[] m, double[] n) {
		for (int i = 0; i < m.length; i++) {
			m[i] = m[i] + n[i];
		}

		return m;
	}

	// This is to calculate the average of vectors
	public static double[] divArray(double[] m, double divisor) {
		for (int i = 0; i < m.length; i++) {
			m[i] = m[i] / divisor;
		}
		return m;
	}

	public static Vector wordToVector(String w, Word2VecModel model) {
		try {
			return model.transform(w);
		} catch (Exception e) {
			return Vectors.zeros(100);
		}
	}

	public static Vector wordToVector(String[] ws, Word2VecModel model) {
		double[] totalSum = wordToVector(ws[0], model).toArray();
		for (int i = 1; i < ws.length; i++) {
			totalSum = sumArray(totalSum, wordToVector(ws[i], model).toArray());

		}
		divArray(totalSum, ws.length);
		return new DenseVector(divArray(totalSum, ws.length));

	}
	
	/* clean the punctuation, stop words and numbers, remove extra space
	 *	and convert to lower case
	 */
	public static String clean(String a){
		if(a != null){
			return a.replaceAll("[^a-zA-Z\\s]", "").replaceAll("\\s+", " ").toLowerCase();
		}else{
			return "";
		}
			
	}

}


--------------

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>Classification</groupId>
	<artifactId>Classification</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<build>
		<sourceDirectory>src</sourceDirectory>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
      			 <plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>exec-maven-plugin</artifactId>
				<version>1.2.1</version>
				<executions>
					<execution>
						<id>gs-poc</id>
						<phase>test</phase>
						<goals>
							<goal>java</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<mainClass>com.infy.gs.automation.client.AlertsDTreeMultiColumns</mainClass>
					<executable>java</executable>
					<arguments>
						<argument>-classpath</argument>
						<argument>target/classes</argument>
						<argument>com.infy.gs.automation.client.AlertsDTreeMultiColumns</argument>
					</arguments>
				</configuration>
			</plugin>  
		</plugins>
	</build>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>
<dependency>
	<groupId>com.google.code.gson</groupId>
	<artifactId>gson</artifactId>
	<version>2.3.1</version>
</dependency>

	
		<dependency>
			<groupId>com.googlecode.json-simple</groupId>
			<artifactId>json-simple</artifactId>
			<version>1.1.1</version>
		</dependency>
		
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-core_2.10</artifactId>
			<version>1.3.1</version>
			<exclusions>
				<exclusion>
					<groupId>org.scala-lang</groupId>
					<artifactId>scala-library</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.10</artifactId>
            <version>1.3.1</version>
        </dependency>
		
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-mllib_2.10</artifactId>
			<version>1.3.1</version>
		</dependency>
		
		<!-- MongoDB -->
		
		  <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>casbah-commons_2.10</artifactId>
        <version>2.8.0</version>
    </dependency>
  <dependency>
	<groupId>org.mongodb</groupId>
	<artifactId>mongo-java-driver</artifactId>
	<version>2.13.0</version>
</dependency>

    
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>casbah-query_2.10</artifactId>
        <version>2.8.0</version>
    </dependency>
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>casbah-core_2.10</artifactId>
        <version>2.8.0</version>
    </dependency>
    <dependency>
        <groupId>de.flapdoodle.embed</groupId>
        <artifactId>de.flapdoodle.embed.mongo</artifactId>
        <version>1.46.4</version>
        <scope>test</scope>
    </dependency>
		<dependency>
	<groupId>org.mongodb</groupId>
	<artifactId>mongo-hadoop-core</artifactId>
	<version>1.3.0</version>
</dependency>
		
			
		
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-streaming_2.10</artifactId>
			<version>1.3.1</version>	
			<exclusions>
				<exclusion>
					<groupId>org.scala-lang</groupId>
					<artifactId>scala-library</artifactId>
				</exclusion>
			</exclusions>		
		</dependency>
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-streaming-kafka_2.10</artifactId>
			<version>1.3.1</version>
		</dependency>

		<dependency>
			<groupId>commons-cli</groupId>
			<artifactId>commons-cli</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>net.sf.jopt-simple</groupId>
			<artifactId>jopt-simple</artifactId>
			<version>3.2</version>
		</dependency>
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>14.0.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-log4j12</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
			</exclusions>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-test</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.testng</groupId>
					<artifactId>testng</artifactId>
				</exclusion>
			</exclusions>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka_2.10</artifactId>
			<version>0.8.1.1</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.zookeeper</groupId>
					<artifactId>zookeeper</artifactId>
				</exclusion>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.4</version>
		</dependency>
<dependency>
	<groupId>org.scala-lang</groupId>
	<artifactId>scala-library</artifactId>
	<version>2.10.4</version>
</dependency>

		<dependency>
			<groupId>com.msiops.footing</groupId>
			<artifactId>footing-tuple</artifactId>
			<version>0.2</version>
		</dependency>

	</dependencies>

</project>


------------

Moving average

package com.infy.gs.automation.client;

import java.io.Serializable;
import java.util.Date;


public class AverageBean implements Serializable{
	
	public Date currenttime;
	public Long resourceruntime;
	public String resourcetype;
	public Float averageusage;
	
	public Date getCurrenttime() {
		return currenttime;
	}
	public void setCurrenttime(Date currenttime) {
		this.currenttime = currenttime;
	}
	public Long getResourceruntime() {
		return resourceruntime;
	}
	public void setResourceruntime(Long resourceruntime) {
		this.resourceruntime = resourceruntime;
	}
	public String getResourcetype() {
		return resourcetype;
	}
	public void setResourcetype(String resourcetype) {
		this.resourcetype = resourcetype;
	}
	public Float getAverageusage() {
		return averageusage;
	}
	public void setAverageusage(Float averageusage) {
		this.averageusage = averageusage;
	}
	public AverageBean(Date currenttime, Long resourceruntime,
			String resourcetype, Float averageusage) {
		super();
		this.currenttime = currenttime;
		this.resourceruntime = resourceruntime;
		this.resourcetype = resourcetype;
		this.averageusage = averageusage;
	}
	
	
}


package com.infy.gs.automation.client;

import java.io.Serializable;

public class AverageCalculator implements Serializable{
	
	  public AverageCalculator(int total, int num)
	  {
		  total_ = total;
		  num_ = num; 
	 }
	  
	  public int total_;
	  public int num_;
	  
	  public float avg() {
		  return total_ / (float) num_;
	  }
	  

}

package com.infy.gs.automation.client;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.spark.HashPartitioner;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.Time;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;


import com.datastax.spark.connector.cql.CassandraConnector;


import static com.datastax.spark.connector.japi.CassandraJavaUtil.*;
import scala.Tuple2;

public class SparkStream implements Serializable{
	

    static String query1;

	public static void main(String args[])
    {
       
		long durationInMilliSec = 30000;
        Map<String,Integer> topicMap = new HashMap<String,Integer>();
        String[] topic = "test,".split(",");
        for(String t: topic)
        {
            topicMap.put("test", new Integer(1));
        }
      
        
        SparkConf _sparkConf = new SparkConf().setAppName("KafkaReceiver").setMaster("local[2]").set("spark.driver.host","zeus07").set("spark.driver.port","8080"); 
       _sparkConf.set("spark.cassandra.connection.host", "localhost");
        
       JavaSparkContext sc = new JavaSparkContext(_sparkConf); 
        
        JavaStreamingContext jsc = new JavaStreamingContext(sc,
				new Duration(durationInMilliSec));
        
					        
        
        JavaPairReceiverInputDStream<String, String> messages = (JavaPairReceiverInputDStream<String, String>) KafkaUtils.createStream(jsc, "localhost:2181","test-consumer-group", topicMap );

        System.out.println("Connection done++++++++++++++");
        JavaDStream<String> data = messages.map(new Function<Tuple2<String, String>, String>() 
                                                {
                                                    public String call(Tuple2<String, String> message)
                                                    {
                                                        return message._2();
                                                    }
                                                }
                                          );
        
        data.print();
   

        
        JavaPairDStream<String, Integer> result = data.mapToPair(
        		 new PairFunction<String, String, Integer>() {
         		    public Tuple2<String, Integer> call(String x) {
         		    	String[] xr = x.split(":");
         		    	return new Tuple2(xr[0], new Integer(xr[1].replaceFirst("%", "").trim()));
         		  }

         		});
        

    
    
    Function<Integer, AverageCalculator> createAcc = new Function<Integer, AverageCalculator>() {
  	  public AverageCalculator call(Integer x) {
  	    return new AverageCalculator(x, 1);
  	  }
  	};
  	
  	Function2<AverageCalculator, Integer, AverageCalculator> addAndCount =
  	  new Function2<AverageCalculator, Integer, AverageCalculator>() {
  	  public AverageCalculator call(AverageCalculator a, Integer x) {
  	    a.total_ += x;
  	    a.num_ += 1;
  	    return a;
  	  }
  	};
  	
  	Function2<AverageCalculator, AverageCalculator, AverageCalculator> combine =
  	  new Function2<AverageCalculator, AverageCalculator, AverageCalculator>() {
  	  public AverageCalculator call(AverageCalculator a, AverageCalculator b) {
  	    a.total_ += b.total_;
  	    a.num_ += b.num_;
  	    return a;
  	  }
  	};
  	
  	AverageCalculator initial = new AverageCalculator(0,0);
  	JavaPairDStream<String, AverageCalculator> avgCounts =
    		result.combineByKey(createAcc, addAndCount, combine, new HashPartitioner(1));
  	
  	
 
  	/*

  	
  	
  	
  	avgCounts.foreach(new Function2<JavaPairRDD<String,AverageCalculator>, Time, Void>() {
  		
		@Override
		public Void call(JavaPairRDD<String, AverageCalculator> values,
				Time time) throws Exception {
			
			values.foreach(new VoidFunction<Tuple2<String, AverageCalculator>> () {

				@Override
				public void call(Tuple2<String, AverageCalculator> tuple)
						throws Exception {
					
					System.out.println("Counter:%%%%%%%%%%%%%%%%%%%%%%" + tuple._1().trim() + "," + tuple._2().avg());
					
										
				}} );
			
			
		}}); 
		
		*/
  	//com.datastax.driver.core.Cluster cluster = Cluster.builder().addContactPoint("localhost").build(); 				     
	//com.datastax.driver.core.Session session = cluster.connect("gs");
  	/*CassandraConnector connector = CassandraConnector.apply(sc.getConf());
  	Session session = connector.openSession();
  	
  	List<JavaPairRDD<String, AverageCalculator>>  averagesRDDList = avgCounts.slice(new Time(System.currentTimeMillis()-durationInMilliSec), new Time(System.currentTimeMillis()));
    	
  	for(JavaPairRDD<String, AverageCalculator> averagesRDD :averagesRDDList){
  		averagesRDD.foreach(new VoidFunction<Tuple2<String, AverageCalculator>> () {

			@Override
			public void call(Tuple2<String, AverageCalculator> tuple)
					throws Exception {
				
				System.out.println("Counter:%%%%%%%%%%%%%%%%%%%%%%" + tuple._1().trim() + "," + tuple._2().avg());
				query1 = "INSERT INTO average(currenttime, resourceruntime, resourcetype, averageusage) VALUES("+
						new Date(System.currentTimeMillis())+","+durationInMilliSec+","+tuple._1().trim()+","+tuple._2().avg()+")";
				
				session.execute(query1); 
									
			}} );
		
	}*/
  		
  		

  
  
  	
    //javaFunctions(avgCounts).writerBuilder("gs", "average", mapToRow(AverageBean.class)).saveToCassandra();

  	
  	
      result.print();
      
      jsc.start();
      jsc.awaitTermination();

  }
    

    
}


________________________-------------------------------
Log saprk


package com.infy.spark.consumer;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.spark.HashPartitioner;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFlatMapFunction;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.Time;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;





import com.google.common.base.Optional;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;

import scala.Tuple2;

import java.io.Serializable;
import java.util.Comparator;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;









import com.infy.spark.utilities.LogPattternMatch;








//import static com.datastax.spark.connector.japi.CassandraJavaUtil.*;
import scala.Tuple2;

public class LogConsumer implements Serializable{


	
	public static void main(String args[])
	{

		long durationInMilliSec = 30000;		
		Map<String,Integer> topicMap = new HashMap<String,Integer>();

		String[] topic = "test,".split(",");
		for(String t: topic)
		{
			topicMap.put("test", new Integer(1));
		}


		SparkConf _sparkConf = new SparkConf().setAppName("KafkaReceiver").setMaster("local[2]").set("spark.driver.host","zeus07").set("spark.driver.port","8080"); 
		_sparkConf.set("spark.cassandra.connection.host", "localhost");

		JavaSparkContext sc = new JavaSparkContext(_sparkConf); 

		JavaStreamingContext jsc = new JavaStreamingContext(sc,	new Duration(durationInMilliSec));	

		 
		JavaPairReceiverInputDStream<String, String> messages = (JavaPairReceiverInputDStream<String, String>) KafkaUtils.createStream(jsc, "127.0.0.1:2181","test-consumer-group", topicMap );       

		System.out.println("Connection done++++++++++++++");
		
		
		

		
		JavaPairDStream<String, String> data  = messages.filter(new Function<Tuple2<String, String>, Boolean> (){

			@Override
			public Boolean call(Tuple2<String, String> arg0) throws Exception {

				//System.out.println("Message "+arg0._2);
				return (new LogPattternMatch().isExceptionMatch(arg0._2));				
			}			
		});	
		

		JavaDStream<String> exceptionMessages = data.map(new Function<Tuple2<String, String>, String>() {

			public String call(Tuple2<String, String> message) {

				System.out.println("A#############################rg)+Arg1***************** "+message._2);
				return (new LogPattternMatch().searchException(message._2));				
			}
		});
		
				
		JavaPairDStream<String, Integer> exceptionKeyValues = exceptionMessages.mapToPair(new PairFunction<String, String, Integer>() {

			@Override
			public Tuple2<String, Integer> call(String arg0) throws Exception {				
				

				System.out.println("888888888888888888888888***************** "+arg0);
				
				return new Tuple2<String, Integer>(arg0,1);
			}
		});

	
		
		JavaPairDStream<String, Integer> exceptionCount = exceptionKeyValues.reduceByKey(new Function2<Integer, Integer, Integer>(){

			@Override
			public Integer call(Integer arg0, Integer arg1) throws Exception {
				
				System.out.println("Arg0+Arg1***************** "+arg0+" "+arg1	);
				
				return arg0+arg1;
			
			}
			
			
		});
		
		
		exceptionCount.print();				
	
		
			
		data.print();
		jsc.start();
		jsc.awaitTermination();

	}
	
	  



}


package com.infy.spark.utilities;

import java.util.LinkedHashMap;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class LogPattternMatch {

	private Map<String,Integer> exceptionCountMap = new LinkedHashMap<String,Integer>();

	public Map<String, Integer> matchAndCount(String line) {

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		while (matcher.find()){

			exceptionName = matcher.group();
			incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println(exceptionName);
		}
		return exceptionCountMap;
	}
	
	public Boolean isExceptionMatch(String line) {
		
		Boolean isMAtch = false;

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		while (matcher.find()){

			exceptionName = matcher.group();
			incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println("______________________________________isExceptionMathch "+exceptionName);
			isMAtch = true;
		}
		//exceptionCountMap.
		return isMAtch;
	}

public String searchException(String line) {
		
		//Boolean isMAtch = false;

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		if(matcher.find()){

			exceptionName = matcher.group();
		//	incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println("Search exception.....................................  "+exceptionName);
		}
		//exceptionCountMap.
		return exceptionName;
	}
	
	public static void main(String[] args) {

		/*String text = "\"Exception in thread \"main\" java.lang.NullPointerException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);"+
				" Exception in thread \"main\" java.lang.ClassCastException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);"+
				" Exception in thread \"main\" java.lang.ClassCastException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);";
*/
		String text = " java.lang.NullPointerException";
		new LogPattternMatch().matchAndCount(text);
	}

	private static void incrementMapCount(Map<String,Integer> map, String exceptionName) {

		Integer currentCount = (Integer)map.get(exceptionName);

		if (currentCount == null) {

			map.put(exceptionName, new Integer(1));
		}
		else {

			currentCount = new Integer(currentCount.intValue() + 1);
			map.put(exceptionName,currentCount);
		}
	}
}

---------------------------------

HOme work............................................

SparkDemo
package com.infy.gs.automation.client;

/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */



/** Java Bean class to be used with the example JavaSqlNetworkWordCount. */
public class JavaRecord implements java.io.Serializable {
	private String alertId;
	private String jobName;
	private String description;
	private String timestamp;
	public String getAlertId() {
		return alertId;
	}
	public void setAlertId(String alertId) {
		this.alertId = alertId;
	}
	public String getJobName() {
		return jobName;
	}
	public void setJobName(String jobName) {
		this.jobName = jobName;
	}
	public String getDescription() {
		return description;
	}
	public void setDescription(String description) {
		this.description = description;
	}
	public String getTimestamp() {
		return timestamp;
	}
	public void setTimestamp(String timestamp) {
		this.timestamp = timestamp;
	}
}
package com.infy.gs.automation.client;

/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

import java.util.HashMap;
import java.util.Map;
import java.util.regex.Pattern;

import com.google.common.collect.Lists;

import org.apache.spark.SparkConf;
import org.apache.spark.SparkContext;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.sql.SQLContext;
import org.apache.spark.sql.DataFrame;
import org.apache.spark.api.java.StorageLevels;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.Time;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;

import scala.Tuple2;

/**
 * Use DataFrames and SQL to count words in UTF8 encoded, '\n' delimited text received from the
 * network every second.
 *
 * Usage: JavaSqlNetworkWordCount <hostname> <port>
 * <hostname> and <port> describe the TCP server that Spark Streaming would connect to receive data.
 *
 * To run this on your local machine, you need to first run a Netcat server
 *    `$ nc -lk 9999`
 * and then run the example
 *    `$ bin/run-example org.apache.spark.examples.streaming.JavaSqlNetworkWordCount localhost 9999`
 */

public final class JavaSqlDataFrame {
  private static final Pattern SPACE = Pattern.compile(" ");

  public static void main(String[] args) {
/*    if (args.length < 2) {
      System.err.println("Usage: JavaNetworkWordCount <hostname> <port>");
      System.exit(1);
    }
*/
	  long durationInMilliSec = 30000;
      Map<String,Integer> topicMap = new HashMap<String,Integer>();
      String[] topic = "test,".split(",");
      for(String t: topic)
      {
          topicMap.put("test", new Integer(1));
      }
    

    // Create the context with a 1 second batch size
    SparkConf sparkConf = new SparkConf().setAppName("JavaSqlNetworkWordCount").setMaster("local[2]").set("spark.driver.allowMultipleContexts", "true").set("spark.driver.host","zeus07").set("spark.driver.port","8080");
    JavaStreamingContext ssc = new JavaStreamingContext(sparkConf, Durations.seconds(3));

    // Create a JavaReceiverInputDStream on target ip:port and count the
    // words in input stream of \n delimited text (eg. generated by 'nc')
    // Note that no duplication in storage level only for running locally.
    // Replication necessary in distributed scenario for fault tolerance.
    //JavaReceiverInputDStream<String> lines = ssc.socketTextStream("Zeus07", 8080, StorageLevels.MEMORY_AND_DISK_SER);
    
    JavaPairReceiverInputDStream<String, String> messages = KafkaUtils.createStream(ssc, "127.0.0.1:2181","test-consumer-group", topicMap );
    /*JavaDStream<String> words = messages.flatMap(new FlatMapFunction<String, String>() {
      @Override
      public Iterable<String> call(String x) {
        return Lists.newArrayList(SPACE.split(x));
      }
    });
    */
    JavaDStream<String> data = messages.map(new Function<Tuple2<String, String>, String>() 
            {
                public String call(Tuple2<String, String> message)
                {
                    return message._2();
                }
            }
      );

data.print();



    // Convert RDDs of the words DStream to DataFrame and run SQL query
    data.foreachRDD(new Function2<JavaRDD<String>, Time, Void>() {
      @Override
      public Void call(JavaRDD<String> rdd, Time time) {
        SQLContext sqlContext = JavaSQLContextSingleton.getInstance(rdd.context());

        // Convert JavaRDD[String] to JavaRDD[bean class] to DataFrame
        JavaRDD<JavaRecord> rowRDD = rdd.map(new Function<String, JavaRecord>() {
          public JavaRecord call(String word) {
            JavaRecord record = new JavaRecord();
            //record.setWord(word);
            
               	String[] xr = word.split(",");
		    	
		    	String alertids[] = xr[0].split(":");
		    	String alertId = alertids[1];
		    	
		    	String jobNames[]=xr[1].split(":");
		    	String jobName=jobNames[1];
		    	
		    	String desc[]=xr[2].split(":");
		    	String description=desc[1];
		    	
		    	String timestamps[]=xr[3].split(":");
		    	String dtimestamp=timestamps[1];
		    	
		    	record.setAlertId(alertId);
		    	record.setJobName(jobName);
		    	record.setDescription(description);
		    	record.setTimestamp(dtimestamp);
		    	
		    	System.out.println("Connection object++++++++++++++"+record);
			

            
            return record;
          }
        });
        DataFrame wordsDataFrame = sqlContext.createDataFrame(rowRDD, JavaRecord.class);

        // Register as table
        wordsDataFrame.registerTempTable("alerts");

        // Do word count on table using SQL and print it
        DataFrame wordCountsDataFrame =
         //   sqlContext.sql("select word, count(*) as total from alerts");
        		
        		sqlContext.sql("SELECT timestamp FROM alerts WHERE jobName = 'CHOG_JOB_NOS'");
        System.out.println("========= " + time + "=========");
        wordCountsDataFrame.show();
        return null;
      }
    });

    ssc.start();
    ssc.awaitTermination();
  }
}

/** Lazily instantiated singleton instance of SQLContext */
class JavaSQLContextSingleton {
  static private transient SQLContext instance = null;
  static public SQLContext getInstance(SparkContext sparkContext) {
    if (instance == null) {
      instance = new SQLContext(sparkContext);
    }
    return instance;
  }
}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>SparkDemo</groupId>
	<artifactId>SparkDemo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<build>
		<sourceDirectory>src</sourceDirectory>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
      			 <plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>exec-maven-plugin</artifactId>
				<version>1.2.1</version>
				<executions>
					<execution>
						<id>gs-poc</id>
						<phase>test</phase>
						<goals>
							<goal>java</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<mainClass>com.infy.gs.automation.client.JavaSqlDataFrame</mainClass>
					<executable>java</executable>
					<arguments>
						<argument>-classpath</argument>
						<argument>target/classes</argument>
						<argument>com.infy.gs.automation.client.JavaSqlDataFrame</argument>
					</arguments>
				</configuration>
			</plugin>  
		</plugins>
	</build>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>


	
		<dependency>
			<groupId>com.googlecode.json-simple</groupId>
			<artifactId>json-simple</artifactId>
			<version>1.1.1</version>
		</dependency>
		
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-core_2.10</artifactId>
			<version>1.3.1</version>
			<exclusions>
				<exclusion>
					<groupId>org.scala-lang</groupId>
					<artifactId>scala-library</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.10</artifactId>
            <version>1.3.1</version>
        </dependency>
		
		
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-streaming_2.10</artifactId>
			<version>1.3.1</version>	
			<exclusions>
				<exclusion>
					<groupId>org.scala-lang</groupId>
					<artifactId>scala-library</artifactId>
				</exclusion>
			</exclusions>		
		</dependency>
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-streaming-kafka_2.10</artifactId>
			<version>1.3.1</version>
		</dependency>

		<dependency>
			<groupId>commons-cli</groupId>
			<artifactId>commons-cli</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>net.sf.jopt-simple</groupId>
			<artifactId>jopt-simple</artifactId>
			<version>3.2</version>
		</dependency>
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>14.0.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-log4j12</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
			</exclusions>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-test</artifactId>
			<version>2.6.0</version>
			<exclusions>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
				<exclusion>
					<groupId>org.testng</groupId>
					<artifactId>testng</artifactId>
				</exclusion>
			</exclusions>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka_2.10</artifactId>
			<version>0.8.1.1</version>
			<exclusions>
				<exclusion>
					<groupId>org.apache.zookeeper</groupId>
					<artifactId>zookeeper</artifactId>
				</exclusion>
				<exclusion>
					<groupId>log4j</groupId>
					<artifactId>log4j</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.4</version>
		</dependency>
<dependency>
	<groupId>org.scala-lang</groupId>
	<artifactId>scala-library</artifactId>
	<version>2.10.4</version>
</dependency>

		<dependency>
			<groupId>com.msiops.footing</groupId>
			<artifactId>footing-tuple</artifactId>
			<version>0.2</version>
		</dependency>

	</dependencies>

</project>

Mongo part:
package com.infy.gs.automation.client;

/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

import java.io.NotSerializableException;
import java.io.Serializable;
import java.net.UnknownHostException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.regex.Pattern;

import org.apache.spark.SparkConf;
import org.apache.spark.SparkContext;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.sql.DataFrame;
import org.apache.spark.sql.SQLContext;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.Time;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;

import com.google.gson.Gson;
import com.mongodb.BasicDBList;
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;
import com.mongodb.DBObject;
import com.mongodb.Mongo;
import com.mongodb.MongoException;
import com.mongodb.util.JSON;

import scala.Tuple2;

/**
 * Use DataFrames and SQL to count words in UTF8 encoded, '\n' delimited text received from the
 * network every second.
 *
 * Usage: JavaSqlNetworkWordCount <hostname> <port>
 * <hostname> and <port> describe the TCP server that Spark Streaming would connect to receive data.
 *
 * To run this on your local machine, you need to first run a Netcat server
 *    `$ nc -lk 9999`
 * and then run the example
 *    `$ bin/run-example org.apache.spark.examples.streaming.JavaSqlNetworkWordCount localhost 9999`
 */

public final class JavaSqlDataFrame implements Serializable{
  

  public static void main(String[] args)  {

	  
	  JavaRecord record = new JavaRecord();
	  
	  
	  
      Map<String,Integer> topicMap = new HashMap<String,Integer>();
      String[] topic = "test,".split(",");
      for(String t: topic)
      {
          topicMap.put("test", new Integer(1));
      }
    

    // Create the context with a 1 second batch size
    SparkConf sparkConf = new SparkConf().setAppName("JavaSqlNetworkWordCount").setMaster("local[2]").set("spark.driver.allowMultipleContexts", "true").set("spark.driver.host","zeus07").set("spark.driver.port","8080");
    JavaStreamingContext ssc = new JavaStreamingContext(sparkConf, Durations.seconds(1));
       
    JavaPairReceiverInputDStream<String, String> messages = KafkaUtils.createStream(ssc, "127.0.0.1:2181","test-consumer-group", topicMap );
    
    JavaDStream<String> data = messages.map(new Function<Tuple2<String, String>, String>() 
            {
                public String call(Tuple2<String, String> message)
                {
                    return message._2();
                }
            }
      );

    data.print();



    // Convert RDDs of the words DStream to DataFrame and run SQL query
    data.foreachRDD(new Function2<JavaRDD<String>, Time, Void>() {
      @Override
      public Void call(JavaRDD<String> rdd, Time time) throws UnknownHostException   {
        //SQLContext sqlContext = JavaSQLContextSingleton.getInstance(rdd.context());
    	  Mongo mongo = new Mongo("localhost", 27017);
    	  DB db = mongo.getDB("sparkdb");
    	  DBCollection alertCollection = null ;
    	  alertCollection = db.getCollection("record");
    	  BasicDBObject basicDBObject = new BasicDBObject();
    	  
         
        // Convert JavaRDD[String] to JavaRDD[bean class] to DataFrame
        JavaRDD<JavaRecord> rowRDD = rdd.map(new Function<String, JavaRecord>() {
          public JavaRecord call(String word)    {
            
            
            
               	String[] xr = word.split(",");
		    	
		    	String alertids[] = xr[0].split("=");
		    	String alertId = alertids[1];
		    	
		    	String jobNames[]=xr[1].split("=");
		    	String jobName=jobNames[1];
		    	
		    	String desc[]=xr[2].split("=");
		    	String description=desc[1];
		    	
		    	String timestamps[]=xr[3].split("=");
		    	String dtimestamp=timestamps[1];
		    	
		    	record.setAlertId(alertId);
		    	record.setJobName(jobName);
		    	record.setDescription(description);
		    	record.setTimestamp(dtimestamp);
		   
		    	
		                   
            return record;
          }
        });
        
       List<JavaRecord> l=rowRDD.collect();
       
       
       for (int i = 0; i < l.size(); i++) {
       	
    	   basicDBObject.put("alertId",l.get(i).getAlertId());
    	   basicDBObject.put("jobName",l.get(i).getJobName()); 
    	   basicDBObject.put("description",l.get(i).getDescription()); 
    	   basicDBObject.put("timestamp",l.get(i).getTimestamp()); 
    	 
				
       }
       if(basicDBObject != null && !basicDBObject.isEmpty())
       alertCollection.insert(basicDBObject);
       
       
       
       /**** Find and display ****/
   	BasicDBObject searchQuery = new BasicDBObject();
   	searchQuery.put("jobName", "CHOG_JOB_NOS");
    
   	DBCursor cursor = alertCollection.find(searchQuery);
    
   	while (cursor.hasNext()) {
   		System.out.println(cursor.next()+"result value by search query");
   	}
    
   	/**** Update ****/
   	// search document where name="mkyong" and update it with new values
   	BasicDBObject query = new BasicDBObject();
   	query.put("jobName", "CHOG_JOB_NOS");
    
   	BasicDBObject newDocument = new BasicDBObject();
   	newDocument.put("jobName", "JOBFAILURE_SEARCH_EXACTPATTERN");
    
   	BasicDBObject updateObj = new BasicDBObject();
   	updateObj.put("$set", newDocument);
    
   	alertCollection.update(query, updateObj);
    
  

        
       /* DataFrame wordsDataFrame = sqlContext.createDataFrame(rowRDD, JavaRecord.class);

        
        wordsDataFrame.registerTempTable("alerts");

        
        DataFrame wordCountsDataFrame =      
        		
        		sqlContext.sql("SELECT timestamp FROM alerts WHERE jobName = 'CHOG_JOB_NOS'");
        System.out.println("========= " + time + "=========");
        wordCountsDataFrame.show();*/
        return null;
      }
    });
    
    
    
    

    ssc.start();
    ssc.awaitTermination();
  }
  
public  static void findEmployee(BasicDBObject query, DBCollection coll ){

	    DBCursor cursor = coll.find(query);

	    try {
	       while(cursor.hasNext()) {
	          DBObject dbobj = cursor.next();
	        //Converting BasicDBObject to a custom Class(Employee)
	          JavaRecord emp = (new Gson()).fromJson(dbobj.toString(), JavaRecord.class);
	          System.out.println(emp.getDescription()+"**********************************");
	       }
	    } finally {
	       cursor.close();
	    }

	}

  
}

/** Lazily instantiated singleton instance of SQLContext */
/*class JavaSQLContextSingleton {
  static private transient SQLContext instance = null;
  static public SQLContext getInstance(SparkContext sparkContext) {
    if (instance == null) {
      instance = new SQLContext(sparkContext);
    }
    return instance;
  }
}*/

Moving average spark
package com.infy.gs.automation.client;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.spark.HashPartitioner;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.Time;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;


import com.datastax.spark.connector.cql.CassandraConnector;


import static com.datastax.spark.connector.japi.CassandraJavaUtil.*;
import scala.Tuple2;

public class SparkStream implements Serializable{
	

    static String query1;

	public static void main(String args[])
    {
       
		long durationInMilliSec = 30000;
        Map<String,Integer> topicMap = new HashMap<String,Integer>();
        String[] topic = "test,".split(",");
        for(String t: topic)
        {
            topicMap.put("test", new Integer(1));
        }
      
        
        SparkConf _sparkConf = new SparkConf().setAppName("KafkaReceiver").setMaster("local[2]").set("spark.driver.host","zeus07").set("spark.driver.port","8080"); 
       _sparkConf.set("spark.cassandra.connection.host", "localhost");
        
       JavaSparkContext sc = new JavaSparkContext(_sparkConf); 
        
        JavaStreamingContext jsc = new JavaStreamingContext(sc,
				new Duration(durationInMilliSec));
        
					        
        
        JavaPairReceiverInputDStream<String, String> messages = (JavaPairReceiverInputDStream<String, String>) KafkaUtils.createStream(jsc, "localhost:2181","test-consumer-group", topicMap );

        System.out.println("Connection done++++++++++++++");
        JavaDStream<String> data = messages.map(new Function<Tuple2<String, String>, String>() 
                                                {
                                                    public String call(Tuple2<String, String> message)
                                                    {
                                                        return message._2();
                                                    }
                                                }
                                          );
        
        data.print();
   

        
        JavaPairDStream<String, Integer> result = data.mapToPair(
        		 new PairFunction<String, String, Integer>() {
         		    public Tuple2<String, Integer> call(String x) {
         		    	String[] xr = x.split(":");
         		    	return new Tuple2(xr[0], new Integer(xr[1].replaceFirst("%", "").trim()));
         		  }

         		});
        

    
    
    Function<Integer, AverageCalculator> createAcc = new Function<Integer, AverageCalculator>() {
  	  public AverageCalculator call(Integer x) {
  	    return new AverageCalculator(x, 1);
  	  }
  	};
  	
  	Function2<AverageCalculator, Integer, AverageCalculator> addAndCount =
  	  new Function2<AverageCalculator, Integer, AverageCalculator>() {
  	  public AverageCalculator call(AverageCalculator a, Integer x) {
  	    a.total_ += x;
  	    a.num_ += 1;
  	    return a;
  	  }
  	};
  	
  	Function2<AverageCalculator, AverageCalculator, AverageCalculator> combine =
  	  new Function2<AverageCalculator, AverageCalculator, AverageCalculator>() {
  	  public AverageCalculator call(AverageCalculator a, AverageCalculator b) {
  	    a.total_ += b.total_;
  	    a.num_ += b.num_;
  	    return a;
  	  }
  	};
  	
  	AverageCalculator initial = new AverageCalculator(0,0);
  	JavaPairDStream<String, AverageCalculator> avgCounts =
    		result.combineByKey(createAcc, addAndCount, combine, new HashPartitioner(1));
  	
  	
 
  	/*

  	
  	
  	
  	avgCounts.foreach(new Function2<JavaPairRDD<String,AverageCalculator>, Time, Void>() {
  		
		@Override
		public Void call(JavaPairRDD<String, AverageCalculator> values,
				Time time) throws Exception {
			
			values.foreach(new VoidFunction<Tuple2<String, AverageCalculator>> () {

				@Override
				public void call(Tuple2<String, AverageCalculator> tuple)
						throws Exception {
					
					System.out.println("Counter:%%%%%%%%%%%%%%%%%%%%%%" + tuple._1().trim() + "," + tuple._2().avg());
					
										
				}} );
			
			
		}}); 
		
		*/
  	//com.datastax.driver.core.Cluster cluster = Cluster.builder().addContactPoint("localhost").build(); 				     
	//com.datastax.driver.core.Session session = cluster.connect("gs");
  	/*CassandraConnector connector = CassandraConnector.apply(sc.getConf());
  	Session session = connector.openSession();
  	
  	List<JavaPairRDD<String, AverageCalculator>>  averagesRDDList = avgCounts.slice(new Time(System.currentTimeMillis()-durationInMilliSec), new Time(System.currentTimeMillis()));
    	
  	for(JavaPairRDD<String, AverageCalculator> averagesRDD :averagesRDDList){
  		averagesRDD.foreach(new VoidFunction<Tuple2<String, AverageCalculator>> () {

			@Override
			public void call(Tuple2<String, AverageCalculator> tuple)
					throws Exception {
				
				System.out.println("Counter:%%%%%%%%%%%%%%%%%%%%%%" + tuple._1().trim() + "," + tuple._2().avg());
				query1 = "INSERT INTO average(currenttime, resourceruntime, resourcetype, averageusage) VALUES("+
						new Date(System.currentTimeMillis())+","+durationInMilliSec+","+tuple._1().trim()+","+tuple._2().avg()+")";
				
				session.execute(query1); 
									
			}} );
		
	}*/
  		
  		

  
  
  	
    //javaFunctions(avgCounts).writerBuilder("gs", "average", mapToRow(AverageBean.class)).saveToCassandra();

  	
  	
      result.print();
      
      jsc.start();
      jsc.awaitTermination();

  }
    

    
}

package com.infy.gs.automation.client;

import java.io.Serializable;

public class AverageCalculator implements Serializable{
	
	  public AverageCalculator(int total, int num)
	  {
		  total_ = total;
		  num_ = num; 
	 }
	  
	  public int total_;
	  public int num_;
	  
	  public float avg() {
		  return total_ / (float) num_;
	  }
	  

}

package com.infy.gs.automation.client;

import java.io.Serializable;
import java.util.Date;


public class AverageBean implements Serializable{
	
	public Date currenttime;
	public Long resourceruntime;
	public String resourcetype;
	public Float averageusage;
	
	public Date getCurrenttime() {
		return currenttime;
	}
	public void setCurrenttime(Date currenttime) {
		this.currenttime = currenttime;
	}
	public Long getResourceruntime() {
		return resourceruntime;
	}
	public void setResourceruntime(Long resourceruntime) {
		this.resourceruntime = resourceruntime;
	}
	public String getResourcetype() {
		return resourcetype;
	}
	public void setResourcetype(String resourcetype) {
		this.resourcetype = resourcetype;
	}
	public Float getAverageusage() {
		return averageusage;
	}
	public void setAverageusage(Float averageusage) {
		this.averageusage = averageusage;
	}
	public AverageBean(Date currenttime, Long resourceruntime,
			String resourcetype, Float averageusage) {
		super();
		this.currenttime = currenttime;
		this.resourceruntime = resourceruntime;
		this.resourcetype = resourcetype;
		this.averageusage = averageusage;
	}
	
	
}
Log consumer

package com.infy.spark.consumer;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.spark.HashPartitioner;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFlatMapFunction;
import org.apache.spark.api.java.function.PairFunction;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.Time;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;





import com.google.common.base.Optional;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.streaming.Duration;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;

import scala.Tuple2;

import java.io.Serializable;
import java.util.Comparator;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;









import com.infy.spark.utilities.LogPattternMatch;








//import static com.datastax.spark.connector.japi.CassandraJavaUtil.*;
import scala.Tuple2;

public class LogConsumer implements Serializable{


	
	public static void main(String args[])
	{

		long durationInMilliSec = 30000;		
		Map<String,Integer> topicMap = new HashMap<String,Integer>();

		String[] topic = "test,".split(",");
		for(String t: topic)
		{
			topicMap.put("test", new Integer(1));
		}


		SparkConf _sparkConf = new SparkConf().setAppName("KafkaReceiver").setMaster("local[2]").set("spark.driver.host","zeus07").set("spark.driver.port","8080"); 
		_sparkConf.set("spark.cassandra.connection.host", "localhost");

		JavaSparkContext sc = new JavaSparkContext(_sparkConf); 

		JavaStreamingContext jsc = new JavaStreamingContext(sc,	new Duration(durationInMilliSec));	

		 
		JavaPairReceiverInputDStream<String, String> messages = (JavaPairReceiverInputDStream<String, String>) KafkaUtils.createStream(jsc, "127.0.0.1:2181","test-consumer-group", topicMap );       

		System.out.println("Connection done++++++++++++++");
		
		
		

		
		JavaPairDStream<String, String> data  = messages.filter(new Function<Tuple2<String, String>, Boolean> (){

			@Override
			public Boolean call(Tuple2<String, String> arg0) throws Exception {

				//System.out.println("Message "+arg0._2);
				return (new LogPattternMatch().isExceptionMatch(arg0._2));				
			}			
		});	
		

		JavaDStream<String> exceptionMessages = data.map(new Function<Tuple2<String, String>, String>() {

			public String call(Tuple2<String, String> message) {

				System.out.println("A#############################rg)+Arg1***************** "+message._2);
				return (new LogPattternMatch().searchException(message._2));				
			}
		});
		
				
		JavaPairDStream<String, Integer> exceptionKeyValues = exceptionMessages.mapToPair(new PairFunction<String, String, Integer>() {

			@Override
			public Tuple2<String, Integer> call(String arg0) throws Exception {				
				

				System.out.println("888888888888888888888888***************** "+arg0);
				
				return new Tuple2<String, Integer>(arg0,1);
			}
		});

	
		
		JavaPairDStream<String, Integer> exceptionCount = exceptionKeyValues.reduceByKey(new Function2<Integer, Integer, Integer>(){

			@Override
			public Integer call(Integer arg0, Integer arg1) throws Exception {
				
				System.out.println("Arg0+Arg1***************** "+arg0+" "+arg1	);
				
				return arg0+arg1;
			
			}
			
			
		});
		
		
		exceptionCount.print();				
	
		
			
		data.print();
		jsc.start();
		jsc.awaitTermination();

	}
	
	  



}

package com.infy.spark.utilities;

import java.util.LinkedHashMap;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class LogPattternMatch {

	private Map<String,Integer> exceptionCountMap = new LinkedHashMap<String,Integer>();

	public Map<String, Integer> matchAndCount(String line) {

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		while (matcher.find()){

			exceptionName = matcher.group();
			incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println(exceptionName);
		}
		return exceptionCountMap;
	}
	
	public Boolean isExceptionMatch(String line) {
		
		Boolean isMAtch = false;

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		while (matcher.find()){

			exceptionName = matcher.group();
			incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println("______________________________________isExceptionMathch "+exceptionName);
			isMAtch = true;
		}
		//exceptionCountMap.
		return isMAtch;
	}

public String searchException(String line) {
		
		//Boolean isMAtch = false;

		String pattern = "\\s([a-zA-Z.]*\\.[a-zA-Z.]*Exception)";
		//String pattern1 = "((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*\\r(?:(.*Exception.*(\\r.*)(\\tat.*\\r)+)))|((?:[a-zA-Z]{3} \\d{1,2}, \\d{4,4} \\d{1,2}:\\d{2}:\\d{2} (AM|PM) (\\(INFO\\)|\\(SEVERE\\)|\\(WARNING\\))).*)";

		Pattern patterns = Pattern.compile(pattern,Pattern.MULTILINE);
		Matcher matcher = patterns.matcher(line);

		String exceptionName = "";

		if(matcher.find()){

			exceptionName = matcher.group();
		//	incrementMapCount(exceptionCountMap , exceptionName);
			System.out.println("Search exception.....................................  "+exceptionName);
		}
		//exceptionCountMap.
		return exceptionName;
	}
	
	public static void main(String[] args) {

		/*String text = "\"Exception in thread \"main\" java.lang.NullPointerException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);"+
				" Exception in thread \"main\" java.lang.ClassCastException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);"+
				" Exception in thread \"main\" java.lang.ClassCastException"+
				"at com.example.myproject.Book.getTitle(Book.java:16)"+
				"at com.example.myproject.Author.getBookTitles(Author.java:25)"+
				"at com.example.myproject.Bootstrap.main(Bootstrap.java:14);";
*/
		String text = " java.lang.NullPointerException";
		new LogPattternMatch().matchAndCount(text);
	}

	private static void incrementMapCount(Map<String,Integer> map, String exceptionName) {

		Integer currentCount = (Integer)map.get(exceptionName);

		if (currentCount == null) {

			map.put(exceptionName, new Integer(1));
		}
		else {

			currentCount = new Integer(currentCount.intValue() + 1);
			map.put(exceptionName,currentCount);
		}
	}
}


kafka 

package com.infy.kafka.start;

import kafka.producer.Partitioner;

public class KafkaSimplePartitioner implements Partitioner {

	public int partition(Object key, int a_numPartitions) {

		
		System.out.println((int) System.currentTimeMillis());
		return (int) System.currentTimeMillis();
		
		
	}
	
	public static void main(String[] args) {
		
		System.out.println((int) System.currentTimeMillis());
		
	}
}

package com.infy.kafka.start;

import java.sql.Timestamp;
import java.util.Date;
import java.util.Properties;
import java.util.Random;

import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;

import com.infy.kafka.values.KafkaProducerConstants;
/**
 * 
 * @author Achint_Verma
 *
 */
public class KafkaStart {

	public static void main(String[] args) {

		Producer<String, String> producer = null;

		try {

			Random rnd = new Random();

			Properties props = new Properties();

			props.put("metadata.broker.list", KafkaProducerConstants.KAFKFA_CLUSTERS);// defines where the Producer can find a one or more Brokers to determine the Leader for each topic
			props.put("serializer.class", "kafka.serializer.StringEncoder");
			//props.put("partitioner.class", "example.producer.SimplePartitioner"); //used to determine which Partition in the Topic the message is to be sent to. Random partition if not specified.
			props.put("request.required.acks", "1");//0,1 or -1 : whether acknowledgement required or not 

			ProducerConfig config = new ProducerConfig(props);
			producer = new Producer<String, String>(config);

			while(true) {

				for (long nEvents = 0; nEvents < KafkaProducerConstants.NO_OF_MESSAGES; nEvents++) {

					String msg = "";
					String msg1 = "";
					Integer alertId = 1;
					String jobName = "";
					String description="";
					Date d=new Date();
					try {

						alertId = rnd.nextInt(100);
						jobName = "CHOG_JOB_NOS";
						description="asdfghjk ";
					
						msg = "AlertId="+alertId+",Jobname="+jobName+",Description="+description+",Timestamp="+ new Timestamp(d.getTime());
								
					
						//msg1 = "Current Memory Usage:" + memoryUsage+"%";

						System.out.println(msg);
						//System.out.println(msg1+"\n");

						KeyedMessage<String, String> data = new KeyedMessage<String, String>("test",msg);
						//KeyedMessage<String, String> data1 = new KeyedMessage<String,String>("test",msg1);
						producer.send(data);
						//producer.send(data1);

						//
						//System.out.println("Sleeping for "+KafkaProducerConstants.SLEEP_TIME+" ms ...");
						//Thread.sleep(KafkaProducerConstants.SLEEP_TIME);
					}
					catch (Exception e) {

						e.printStackTrace();
						System.err.println("Error in sending mesage :"+ msg);
					}
				}
				Thread.sleep(KafkaProducerConstants.SLEEP_TIME);
			}
		}
		catch (Exception e) {

			System.err.println("Some Exception occured:");
			e.printStackTrace();
		}
		finally{

			producer.close();
		}
	}
}

package com.infy.kafka.values;

public class KafkaProducerConstants {
	
	public static final long NO_OF_MESSAGES = 3;
	public static final int SLEEP_TIME = 10000;
	public static final String KAFKFA_CLUSTERS = "localhost:9092"; 
}

kafkalogprovider
package com.infy.kafka.logprovider;

import java.io.File;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import org.apache.commons.io.input.Tailer;
import org.apache.commons.io.input.TailerListener;

import com.infy.kafka.values.KafkaProducerConstants;

/**
 * For reading log file and sending to Kafka Broker
 * 
 * @author achint_verma
 *
 */

public class KafkaLogReader {

	// private final Log logger = LogFactory.getLog(KafkaLogReader.class);

	public static void main(String[] args) {

		String filePath = KafkaProducerConstants.INPUT_LOG_LOCATION;

		if (!new File(filePath).exists()) {

			System.err.println(filePath + " is not found !!!! . Please check INPUT_LOG_LOCATION parameter  ");
			System.exit(0);
		}
		System.out.println("File Exists!");
		TailerListener listener = new KafkaTailListener();
		Tailer tailer = new Tailer(new File(filePath), listener, KafkaProducerConstants.TAIL_CHECK_INTERVAL);
		
		ExecutorService pool = Executors.newCachedThreadPool();
		pool.submit(tailer);
	}
}

package com.infy.kafka.logprovider;

import java.util.LinkedList;
import java.util.List;
import java.util.Properties;

import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;

import org.apache.commons.io.input.Tailer;
import org.apache.commons.io.input.TailerListenerAdapter;

import com.infy.kafka.values.KafkaProducerConstants;

public class KafkaTailListener extends TailerListenerAdapter {	

	private Producer<String, String> producer;	
	private List<KeyedMessage<String, String>> messageList = new LinkedList<KeyedMessage<String,String>>();//for storing batch of messages to be sent to Kakfa producer

	public KafkaTailListener() {		

		Properties props = new Properties();

		props.put("metadata.broker.list", KafkaProducerConstants.KAFKFA_CLUSTERS);// defines where the Producer can find a one or more Brokers to determine the Leader for each topic
		props.put("serializer.class", "kafka.serializer.StringEncoder");		
		props.put("request.required.acks", "1");//0,1 or -1 : whether acknowledgement required or not
		//props.put("partitioner.class", "example.producer.SimplePartitioner"); //used to determine which Partition in the Topic the message is to be sent to. Random partition if not specified.

		ProducerConfig config = new ProducerConfig(props); 
		this.producer = new Producer<String, String>(config);                
	}

	public void init(Tailer tailer) { 
	} 
	/**
	 * For sending NO_OF_MESSAGES lines from log file to Kafka broker
	 * Every new NO_OF_MESSAGES that are written to log file are stored in LinkedList.
	 * Once size of LinkedList reaches NO_OF_MESSAGES, data is pushed to Kafka broker and list is cleared for storing new NO_OF_MESSAGES lines
	 * @param line Line received from tail on log
	 * @ 
	 */
	public void handle(String line) {

		KeyedMessage<String, String> data = new KeyedMessage<String, String>(KafkaProducerConstants.KAFKFA_TOPIC,line);				
		messageList.add(data);
		System.out.println("Adding message ["+line+"] to message batch ...");

		if (messageList.size() == KafkaProducerConstants.NO_OF_MESSAGES) {

			try {	
				System.out.println("... message batch ready !");
				producer.send(messageList);				

				System.out.println("---------------------------------------------------------------------");
				System.out.println("MESSAGE BATCH SENT !!");
				System.out.println("---------------------------------------------------------------------");
				System.out.println("\n");
			} 
			catch (Exception e) {

				e.printStackTrace();
				System.out.println("Error in sending message");
			}
			finally {

				messageList.clear();				
			}			
		}		
	}

	public void handle(Exception ex) { 

		//logger.error("Kafka Tailer ", ex); 
	} 
}

package com.infy.kafka.values;

public class KafkaProducerConstants {
	
	public static final long NO_OF_MESSAGES = 5;
	public static final int SLEEP_TIME = 3000;
	public static final String KAFKFA_CLUSTERS = "localhost:9092";
	public static final int KAFKFA_PORT = 9092;
	public static final String INPUT_LOG_LOCATION = "/root/kafka/log.txt";
	public static final int KAKFA_PARTITION = 0;
	public static final String KAFKFA_TOPIC = "test";
	public static final long TAIL_CHECK_INTERVAL = 3000;
}

kafkaprovider

package com.infy.kafka.start;

import java.util.Properties;
import java.util.Random;

import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;

import com.infy.kafka.values.KafkaProducerConstants;
/**
 * 
 * @author Achint_Verma
 *
 */
public class KafkaStart {

	public static void main(String[] args) {

		Producer<String, String> producer = null;

		try {

			Random rnd = new Random();

			Properties props = new Properties();

			props.put("metadata.broker.list", KafkaProducerConstants.KAFKFA_CLUSTERS);// defines where the Producer can find a one or more Brokers to determine the Leader for each topic
			props.put("serializer.class", "kafka.serializer.StringEncoder");
			//props.put("partitioner.class", "example.producer.SimplePartitioner"); //used to determine which Partition in the Topic the message is to be sent to. Random partition if not specified.
			props.put("request.required.acks", "1");//0,1 or -1 : whether acknowledgement required or not 

			ProducerConfig config = new ProducerConfig(props);
			producer = new Producer<String, String>(config);

			while(true) {

				for (long nEvents = 0; nEvents < KafkaProducerConstants.NO_OF_MESSAGES; nEvents++) {

					String msg = "";
					String msg1 = "";
					String CPUUsage = "";
					String memoryUsage = "";

					try {

						CPUUsage = String.valueOf(rnd.nextInt(100));
						memoryUsage = String.valueOf(rnd.nextInt(100));
						msg = "Current CPU Usage:" + CPUUsage + "%";
						msg1 = "Current Memory Usage:" + memoryUsage+"%";

						System.out.println(msg);
						System.out.println(msg1+"\n");

						KeyedMessage<String, String> data = new KeyedMessage<String, String>("test",msg);
						KeyedMessage<String, String> data1 = new KeyedMessage<String,String>("test",msg1);
						producer.send(data);
						producer.send(data1);

						//
						//System.out.println("Sleeping for "+KafkaProducerConstants.SLEEP_TIME+" ms ...");
						//Thread.sleep(KafkaProducerConstants.SLEEP_TIME);
					}
					catch (Exception e) {

						e.printStackTrace();
						System.err.println("Error in sending mesage :"+ msg);
					}
				}
				Thread.sleep(KafkaProducerConstants.SLEEP_TIME);
			}
		}
		catch (Exception e) {

			System.err.println("Some Exception occured:");
			e.printStackTrace();
		}
		finally{

			producer.close();
		}
	}
}

package com.infy.kafka.start;

import java.util.Properties;
import java.util.Random;

import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;

import com.infy.kafka.values.KafkaProducerConstants;
/**
 * 
 * @author Achint_Verma
 *
 */
public class KafkaStart {

	public static void main(String[] args) {

		Producer<String, String> producer = null;

		try {

			Random rnd = new Random();

			Properties props = new Properties();

			props.put("metadata.broker.list", KafkaProducerConstants.KAFKFA_CLUSTERS);// defines where the Producer can find a one or more Brokers to determine the Leader for each topic
			props.put("serializer.class", "kafka.serializer.StringEncoder");
			//props.put("partitioner.class", "example.producer.SimplePartitioner"); //used to determine which Partition in the Topic the message is to be sent to. Random partition if not specified.
			props.put("request.required.acks", "1");//0,1 or -1 : whether acknowledgement required or not 

			ProducerConfig config = new ProducerConfig(props);
			producer = new Producer<String, String>(config);

			while(true) {

				for (long nEvents = 0; nEvents < KafkaProducerConstants.NO_OF_MESSAGES; nEvents++) {

					String msg = "";
					String msg1 = "";
					String CPUUsage = "";
					String memoryUsage = "";

					try {

						CPUUsage = String.valueOf(rnd.nextInt(100));
						memoryUsage = String.valueOf(rnd.nextInt(100));
						msg = "Current CPU Usage:" + CPUUsage + "%";
						msg1 = "Current Memory Usage:" + memoryUsage+"%";

						System.out.println(msg);
						System.out.println(msg1+"\n");

						KeyedMessage<String, String> data = new KeyedMessage<String, String>("test",msg);
						KeyedMessage<String, String> data1 = new KeyedMessage<String,String>("test",msg1);
						producer.send(data);
						producer.send(data1);

						//
						//System.out.println("Sleeping for "+KafkaProducerConstants.SLEEP_TIME+" ms ...");
						//Thread.sleep(KafkaProducerConstants.SLEEP_TIME);
					}
					catch (Exception e) {

						e.printStackTrace();
						System.err.println("Error in sending mesage :"+ msg);
					}
				}
				Thread.sleep(KafkaProducerConstants.SLEEP_TIME);
			}
		}
		catch (Exception e) {

			System.err.println("Some Exception occured:");
			e.printStackTrace();
		}
		finally{

			producer.close();
		}
	}
}

package com.infy.kafka.values;

public class KafkaProducerConstants {
	
	public static final long NO_OF_MESSAGES = 10;
	public static final int SLEEP_TIME = 30000;
	public static final String KAFKFA_CLUSTERS = "localhost:9092"; 
}
















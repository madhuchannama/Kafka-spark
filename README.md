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

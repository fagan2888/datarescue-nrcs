[AWDB Web Service Tutorial](https://www.wcc.nrcs.usda.gov/web_service/AWDB_Web_Service_Tutorial.htm)

Introduction

What is a Web Service?

AWDB Web Service Overview

Web Service URL

Conventions

Generate Java Client Stubs

Create Instance of AwdbWebService Object

Typical Use Cases

Use Case 1: Get Inventory of Stations

Use Case 2: Get Period of Record for One Station

Use Case 3: Get Last Seven Days of Data

Note: The following tutorial describes how to use a web service to access data in the Natural Resources Conservation Service (NRCS) National Water and Climate Center (NWCC) AWDB database. It is not meant to introduce general concepts and programming requirements relating to web services.  Refer to the AWDB Web Service Reference for a complete listing of all Methods, Classes, Network Codes, and Element Codes.
Introduction

The National Water and Climate Center maintains a large database of soil, water, and climate data known as AWDB (air and water database).  This database includes data from NWCC’s extensive automated data collection system of snowpack and related climate data called SNOTEL (for SNOw TELemetry), its Soil Climate Analysis Network (SCAN) stations, as well as data from snow courses, streamflow stations, reservoirs, climate indices, National Weather Service COOP stations, 30-year normals, water supply forecasts, and more.  All of this data can be accessed publicly through the AWDB Web Service.
What is a Web Service?

A Web Service is a way to communicate between two devices over the Internet.  A user (client) of a Web Service invokes remote procedure calls on a Web Service that provides some data or other services.  The user of a Web Service is typically another application.  The application can be written in any language and on any platform.

A Web Service provider will provide a WSDL (Web Services Description Language) file that fully describes the Web Service.  The WSDL file is an XML document that has all of the information needed to access and use a Web Service. 

The WSDL file describes all of the calls that can be made on the Web Service, the parameters and return values for each call, and the address of the service.  Most programming languages provide a way to translate a WSDL file into code that can be used to make calls to the Web Service.
AWDB Web Service Overview

The AWDB Web Service is a SOAP (Simple Object Access Protocol) Web Service.  The AWDB Web Service provides methods for accessing metadata (about elements, units, stations, etc.), data, 30-year normals, and water supply forecasts.

To access the AWDB Web Service from a Java project, you must create the Java stubs from the Web Services Description Language (WSDL) and include the classes in your project. 

This tutorial describes how to:

- Generate the Java client stubs from the WSDL.
- Create an instance of the AwdbWebService object.
- Make calls to the AWDB database. Several “use cases” show examples of how to extract different types of data for different timeframes.

Web Service URL

The URL for the WSDL for the AWDB Web Service is: https://www.wcc.nrcs.usda.gov/awdbWebService/services?WSDL
Conventions

The following conventions apply to the AWDB Web Service:

- Method parameters marked with an asterisk (*) are required.
- Several methods take a station triplet string or a list of triplets.  These triplets identify a station and are composed of a three part string that contains the station id, the state code, and the network code of the station all delimited by a colon (‘:’), as follows:
    ```
    [station id]:[state code]:[network code]
    ```
- Example:  "302:OR:SNTL" 
- Methods that request results for multiple stations will return an array holding the results in the same order the stations were given.
- Dates must be in the format YYYY-MM-DD.  For example January 4, 2013 would be formatted as 2013-01-04.

Generate Java Client Stubs

wsimport is the utility to generate Java client stubs from a WSDL. It is supplied with the Java JDK and is located in the bin directory.   If you do not have the JDK, you will need to download it first. 

The wsimport utility simply needs to be given the desired package in which to put the new stubs and the WSDL address.  From a command line, use the following command:

```
wsimport -p gov.usda.nrcs.wcc.awdbWebService -s . ‘https://www.wcc.nrcs.usda.gov/awdbWebService/services?WSDL’
```

This generates the stubs for the AWDB Web Service and places them in the package gov.usda.nrcs.wcc.awdbWebService.  Once the client stubs are generated, they are imported and used just like any other Java package.
Create Instance of AwdbWebService Object

To make calls to the AWDB Web Service, you need an instance of the AwdbWebService class that is pointed to the Web Service.   This can be done once prior to using the AWDB Web Service calls, such as on program start, and the instance can be reused repeatedly throughout the application. 

To create this instance, use the following code:

```java
import gov.usda.nrcs.wcc.awdbWebService.*;      //The web service stubs
import javax.xml.namespace.QName;
import java.net.URL;

AwdbWebService m_webService = null;
try
{
    URL wsURL = newURL("https://www.wcc.nrcs.usda.gov/awdbWebService/services?wsdl");
    AwdbWebService_Service lookup = new AwdbWebService_Service(wsURL, new
    QName("https://www.wcc.nrcs.usda.gov/ns/awdbWebService", "AwdbWebService"));
        m_webService = lookup.getAwdbWebServiceImplPort();
}
catch (Exception e)
{
    //On failure do...
}
```

Now you can use m_webService to make calls to the AWDB database. For example:  m_webService.getElements() returns all elements in the AWDB database.
Typical Use Cases

Following is an example java class that contains separate methods which demonstrate three common uses of the AWDB Web Service:

Use Case 1: Get an inventory of stations. The example gets an inventory of all SNOTEL stations in Oregon that have snow-water equivalent data (element code WTEQ) and returns a list of stations.

Use Case 2: Get period of record data. The example returns period of record data that are snow-water equivalent for a given station and date range.

Use Case 3: Get last seven days data. The example returns the last seven days of snow-water equivalent data, relative to today, for a given station.

The main method utilizes Use Case 1 and Use Case 3 by looping through the stations and returning each station’s metadata and values.

```java
import gov.usda.nrcs.wcc.awdbWebService.*;
import javax.xml.namespace.QName;
import java.net.URL;
import java.math.BigDecimal;
import java.text.SimpleDateFormat;
import java.util.*;

public class WSUseCases
{
    static final SimpleDateFormat dateFormat = new SimpleDateFormat(
        "yyyy-MM-dd");
    AwdbWebService m_webService = null;

/**
 * Constructor which initializes a web service instance.
 */

public WSUseCases()
{
    initWebService();
}   

/**
 * Initialize the web service member variable m_webService.
 */

private void initWebService()
{
    try
    {
        URL wsURL = new URL
            "https://www.wcc.nrcs.usda.gov/awdbWebService/services?wsdl");

        AwdbWebService_Service lookup = new AwdbWebService_Service(wsURL,
            new QName("https://www.wcc.nrcs.usda.gov/ns/awdbWebService",
                "AwdbWebService"));
        m_webService = lookup.getAwdbWebServiceImplPort():
    }
    catch (Exception e)
    {   
        System.out.println("Problem creating web service client: "
            + e.getMessage());   

    }   
}   

 /**
    * Example main program entry point which:
    * 1 - instantiates this class and creates a web service object
    * 2 - gets a list of stations, loop through the stations and print out each
    *     station's metadata
    * 3 - uses Use Case 3 to get last 7 days data for each station
    *     and print out each day's date and values.
    * @param args
    */

    public static void main(String[] args){
        WSUseCases useCase = new WSUseCases();

        //Use Case 1: Get Inventory of Stations
        // NOTE:  For this tutorial, the selection criteria of
        // network, state and element  are hard-coded
        // as ‘SNTL’, ’OR’ and ’WTEQ’ in the getData function.
        // NOTE: The getStationMetadata*  function
        // may return some sites that do not have data
        // the date range you are interested in. 
        // You can remove those stations by checking their
        // begin and end dates before retrieving data.

        List<StationMetaData> metadata = useCase.getStations();

        String stationName, stationTriplet, beginDate, endDate, countyName,
        huc, shefID, fipsCountryCode, fipsCountyCode, fipsStateNumber;

        BigDecimal elevation, latitude, longitude, stationDataTimeZone;

        // Loop through list of stations and retrieve and
        // print out metadata for one station

        for (StationMetaData metadatum : metadata)
        {
            stationName = metadatum.getName();
            stationTriplet = metadatum.getStationTriplet();
            beginDate = metadatum.getBeginDate();
            endDate = metadatum.getEndDate();
            countyName = metadatum.getCountyName();
            elevation = metadatum.getElevation();
            huc = metadatum.getHuc();
            latitude = metadatum.getLatitude();
            longitude = metadatum.getLongitude();
            shefID = metadatum.getShefId();
            stationDataTimeZone = metadatum.getStationDataTimeZone();
            fipsCountryCode = metadatum.getFipsCountryCd();
            fipsCountyCode = metadatum.getFipsCountyCd();
            fipsStateNumber = metadatum.getFipsStateNumber();

        System.out.println("Station Name:       " + stationName + '\n' +
                           "Station Triplet:    " + stationTriplet + '\n' +
                           "Begin Date:         " + beginDate + '\n' +
                           "End Date:           " + endDate + '\n' +
                           "County Name:        " + countyName + '\n' +
                           "Elevation:          " + elevation + '\n' +
                           "HUC:                " + huc + '\n' +
                           "Latitude:           " + latitude + '\n' +
                           "Longitude:          " + longitude + '\n' +
                           "SHEF ID:            " + shefID + '\n' +
                           "Station Time Zone:  " + stationDataTimeZone + '\n' +
                           "Fips Country Code:  " + fipsCountryCode + '\n' +
                           "Fips County Code:   " + fipsCountyCode + '\n' +
                           "Fips State #:       " + fipsStateNumber);

            /*
             * Use Case 3: Get Last 7 Days Data for One Station
             * This will get DAILY data for each of the stations from
             * Use Case 1.
             * Note: To apply Use Case 2: Period of Record of Data for One Station,
             * see example method getPeriodOfRecord().
             */

            Data[] data = useCase.getLastSevenDaysData(stationTriplet);
            String beginDt = data[0].getBeginDate();
            String flags = data[0].getFlags().toString();
            BigDecimal[] stationValues = data[0].getValues().toArray(
                new BigDecimal[0]);
            // Get values for current station
            for (int i = 0; i <= stationValues.length; i++)
            {
                System.out.println("Date: Flags/Value: " + beginDt + ":  "
                    + flags + "/" + stationValues[i] + '\n');
            }
        }
    }
```
Use Case 1: Get Inventory of Stations

```java
 /**
     * Use Case 1: Get Inventory of Stations
     * This example will get an inventory of all stations in Oregon
     * for SNOTEL stations that have Snow Water Equivalent
     * and return  list of stations   
     */

    public List<StationMetaData> getStations(){
        List<String> stationIds = null;
        List<String> stateCds = null;
        List<String> networkCds = Arrays.asList("SNTL");
        List<String> hucs = null;
        List<String> countyNames = null;
        BigDecimal minLatitude = null;
        BigDecimal maxLatitude = null;
        BigDecimal minLongitude = null;
        BigDecimal maxLongitude = null;
        BigDecimal minElevation = null;
        BigDecimal maxElevation = null;
        List<String> elementCodes = Arrays.asList("WTEQ");
        List<Integer> ordinals = Arrays.asList(1);
        List<HeightDepth> heightDepths = null;

        /*
         * If (logicalAnd) is true, the getStations() call will return only
         * stations that match ALL of the parameters passed in, otherwise it’ll
         * return stations that match ANY of the parameters passed in.
         */       

        boolean logicalAnd = true;
        List<String> stationTriplets = m_webService.getStations(stationIds,
            stateCds, networkCds, hucs, countyNames, minLatitude,
            maxLatitude, minLongitude, maxLongitude, minElevation,
            maxElevation, elementCodes, ordinals, heightDepths, logicalAnd);
        List<StationMetaData> stations = m_webService
            .getStationMetadataMultiple(stationTriplets)
        return stations;
    }
```

Use Case 2: Get Period of Record for One Station

```java
/**
     * Use Case 2: Get period of record for one station.
     * This will return period of Data that are SNOW WATER EQUIVALENT (element
     * code = WTEQ) for a given station and date range.
     * Note: Always use an ordinal of 1, and heightDepth of null
     * (height depth is only used for soil sensors)
     * @param p_stationTriplet The station to get data for, ex: "471:ID:SNTL"
     * @param p_beginDate The begin date - a String format "yyyy-MM-dd"
     * @param p_endDate The end date - a String format "yyyy-MM-dd"
     * @return An Array of Data Objects
     */

    public Data[] getPeriodOfRecord(String p_stationTriplet, String p_beginDate,     String p_endDate){               
        Data[] values = m_webService.getData(Arrays.asList(p_stationTriplet),
            "WTEQ", 1, null, Duration.DAILY, true, p_beginDate, p_endDate, true)
            .toArray(new Data[0]);
        return values;
    }
```

Use Case 3: Get Last Seven Days of Data

```java
 /**
     * Use Case 3: Get last seven days of data.
     * This will return the last seven days of SNOW WATER EQUIVALENT (element
     * code = WTEQ data, relative to today, for a given station.
     * Note: Always use an ordinal of 1, and heightDepth of null
     * (height depth is only used for soil sensors)    
     * @param p_stationTriplet The station to get data for, ex: "471:ID:SNTL"
     * @return An Array of Data Objects
     */

    public Data[] getLastSevenDaysData(String p_stationTriplet){

        String today = dateFormat.format(new Date());
        Calendar lastWeek = GregorianCalendar.getInstance();
        lastWeek.add(Calendar.DAY_OF_YEAR, -7);
        String sevenDaysAgo = dateFormat.format(lastWeek.getTime());

        Data[] values = m_webService.getData(Arrays.asList(p_stationTriplet),
            "WTEQ", 1, null, Duration.DAILY, true, sevenDaysAgo, today, ture)
            .toArray(new Data[0]);

        return values;
     }

/*  End class WSUseCases */

}
```

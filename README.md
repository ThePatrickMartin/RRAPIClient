# RRAPIClient
## Table of contents
* [Overview](#overview)
* [Usage](#usage)

## Overview
C++ DLL project in VS 2019 for calling RadioReference.com SOAP API.

Uses these source files from the [Ginivia gSoap](https://www.genivia.com/doc/guide/html/index.html) library:
* dom.cpp
* stdsoap2.h
* stdsoap2.cpp

The following source files were generated directly from http://api.radioreference.com/soap2?wsdl using the gsoap code generator tools, wsdl2h and soapcpp2:
* rrapi_C.api
* rrapi_Stub.h
* rrapi_RRWsdlBindingProxy.cpp
* rrapi_RRWsdlBindingProxy.h

The following gsoap-specific preprocessor directives MUST be defined for the project:
* WITH_NONAMESPACES
* SOAP_STD_EXPORTS

## Workaround for RadioReference's incorrect WSDL
As of 12/31/2020, some method responses from the RadioReference.com API may violate the type definitions of their own WSDL. 
For example, the TalkgroupCat type is defined in the WSDL thus:
```
 <xsd:complexType name="TalkgroupCat">
  <xsd:all>
   <xsd:element name="tgCid" type="xsd:int" documentation="Test Doco"/>
   <xsd:element name="sid" type="xsd:int"/>
   <xsd:element name="tgCname" type="xsd:string"/>
   <xsd:element name="tgSort" type="xsd:int"/>
   <xsd:element name="tgSortBy" type="xsd:int"/>
   <xsd:element name="lat" type="xsd:decimal"/>
   <xsd:element name="lon" type="xsd:decimal"/>
   <xsd:element name="range" type="xsd:decimal"/>
   <xsd:element name="lastUpdated" type="xsd:dateTime"/>
  </xsd:all>
 </xsd:complexType>
```
But the API may return items of this type with nil element values:
```
    <item xsi:type="tns:TalkgroupCat">
       <tgCid xsi:type="xsd:int">15751</tgCid>
       <sid xsi:type="xsd:int">6766</sid>
       <tgCname xsi:type="xsd:string">Wylie Police</tgCname>
       <tgSort xsi:type="xsd:int">30</tgSort>
       <tgSortBy xsi:nil="true" xsi:type="xsd:int"/>
       <lat xsi:type="xsd:decimal">33.02</lat>
       <lon xsi:type="xsd:decimal">-96.54</lon>
       <range xsi:type="xsd:decimal">5</range>
       <lastUpdated xsi:type="xsd:dateTime">2015-05-08T10:50:38-05:00</lastUpdated>
    </item>
```
As a workaround, I defined a new preprocessor directive **WITH_IMPLICIT_NILLABLE** which will cause gsoap's XML validator to ignore nil elements in the RadioReference API response even if those types were not defined in the WSDL with the "nillable=true" attribute

## Usage
First, follow [these guidelines](https://docs.microsoft.com/en-us/cpp/build/walkthrough-creating-and-using-a-dynamic-link-library-cpp?view=msvc-160#to-create-a-client-app-in-visual-studio) for creating a console application which references a DLL
<br/>
### Code Example
```
#include <iostream>
#include <rrapi_RRWsdlBindingProxy.h>

int main()
{
	RRWsdlBindingProxy rrapi;

	//API Authorization - requires Premium account and API key
	ns1__authInfo auth;
	auth.username = "your_userid";
	auth.password = "your_password";
	auth.appKey = "your_key";

	ns1__getCountryListResponse resp;

	//Get list of countries
	int err = rrapi.getCountryList(resp);
	if (err == SOAP_OK) {
		//Output country names
		int iCount = resp.return_->__size;
		for (int iIdx = 0; iIdx < iCount; iIdx++) {
			std::cout <<  resp.return_->__ptr[iIdx]->countryName << "\n";
		}
	}
	else {
		rrapi.soap_stream_fault(std::cerr);
	}
	
    rrapi.destroy();
}
```

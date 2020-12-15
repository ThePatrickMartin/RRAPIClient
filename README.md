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

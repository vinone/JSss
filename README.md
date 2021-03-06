# JSss

Nodejs Amazon S3 Multipart upload module

[![Build Status](https://secure.travis-ci.org/TotenDev/JSss.png?branch=master)](http://travis-ci.org/TotenDev/JSss)

## Requirements

- [node](https://github.com/joyent/node)
- [npm](https://github.com/isaacs/npm)
- [node-aws-sign](https://github.com/egorFiNE/node-aws-sign) (signs AWS requests)
- [node-xml2json](https://github.com/buglabs/node-xml2json) (decode AWS requests)

## Installation

Download and install dependencies

    $ npm install

## Usage

    var MultiPart = require("./src/JSss.js")("ohByBucket","MyAccessKey","mySecret","folde/theFileName.zip");
    //Register for end event
	MultiPart.on("jsss-end",function () {
		console.log("end");	
	});
	//Register for error event
	MultiPart.on("jsss-error",function (err) {
		console.log(err);
	});
	//Upload successeded or finished
	MultiPart.on("jsss-upload-notice",function (partNumber,status) {
		if (status) {
			partFinished++;
			if (partFinished == partCount) {
				MultiPart.finishUpload();
			}
		}
		else {
			//try again ??
		}
	});
	//Must be registered to MultiPart API start
	MultiPart.on("jsss-ready",function () {
		console.log("ready");
	
		partFinished = 0;
		partCount = 2;
		//All datas need to be 5MB>
		MultiPart.uploadChunk("the big data",1);
		MultiPart.uploadChunk("the big data2",2);
	});

More samples at `samples/` directory.

## Methods

#### Initialize Wrapper

Parameters:

* bucketID - **Type:**string - **Description:**Name of Object in S3 bucket   - **REQUIRED**
* AWSAccessKeyID - **Type:**string - **Description:**AWS AccessKeyID - **REQUIRED**
* AWSSecretAccessKey - **Type:**string - **Description:**AWS SecretAccessKey - **REQUIRED**
* fileName - **Type:**string - **Description:**fileName to be on S3 - **REQUIRED**
* options - **Type:**OptionObject - **Description:**Options Object - **OPTIONAL**
* options.endPoint - **Type:**string - **Description:**End point to be used, default `s3.amazonaws.com` - **OPTIONAL**
* options.useSSL - **Type:**boolean - **Description:**Use SSL or not, default is true - **OPTIONAL**

Sample:

    var MultiPart = require("./src/JSss.js")("myBucket","AWSAccessKey","AWSSecretAccessKey","fileNameToBeUp",{ endPoint:"secondary.s3.com",useSSL:false });
    
---
#### Upload Chunk

Notice this function will not call error listener, it will call upload-notice listener with positionChuck parameter and if succeeded or not, so you can try to re-upload that part if you want.

Parameters:
- chunkData - **Type:**string || Buffer - **Description:**Chunk to be uploaded - **REQUIRED**
- chunkPosition - **Type:**number - **Description:**Chunk Position, so you can upload multiple parts at same time - **REQUIRED**
- fileEncoding - **Type:**number - **Description:**Which encoding to use when uploading. Default is `utf8` - **OPTIONAL**

Sample:

    MultiPart.uploadChunk(chunkData,chunkPosition,'binary');
    
---
#### Finish Upload
This method will finish upload, and can take a bit long for large files, since amazon will only answer the request when all parts are together.

Sample:

    MultiPart.finishUpload();

---
#### Finish Upload
This method will cancel upload, and delete all uploaded chunks.

Sample:

    MultiPart.abortUpload();


## Events

####Ready 
This event **MUST** be registered in order to wrapper start. When this event is reached you are able to start uploading chunks.

Event-String: `jsss-ready`

Sample:

    //Must be registered to MultiPart API start
	MultiPart.on("jsss-ready",function () {
		console.log("I'm ready :)");
	}
---
####Upload Notice
This event will be reached when an upload succeeded or failed.

Event-String: `jsss-upload-notice`

Sample:

    MultiPart.on("jsss-upload-notice",function (partNumber,status) {
	    if (status) {
	       console.log("success :) on part:" + partNumber);
	    }else {
		   console.log("error on part:" + partNumber + "let's try again?");
	    }
	});
---
####Error
This event will be reached when an error occur in any fundamental part of upload (`start`,`finish`,`abort`).
Do **NOT** call `terminate` or `abort` method from error event, since those methods can emit an error event.

Event-String: `jsss-error`

Sample:

	MultiPart.on("jsss-error",function (err) {
		console.log("Bad",err);
	}
---
####End
This event will be reached when upload finished by `abortUpload()` or `finishUpload()` OR if it didn't start properly.

Event-String: `jsss-end`

Sample:

	MultiPart.on("jsss-end",function () {
		console.log("Bye");
	}

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## License

[MIT](JSss/raw/master/LICENSE)
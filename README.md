# SUREFHIR Demo R4 Version 1.0

This demo shows how incoming messages of different formats (HL7, XML, or CDA) can be transformed into SDA and ulitmately sent out via a FHIR® server Interop operation.

## Contents

### Script files
 1. `DockerFile` — script for building a Docker container
 2. `Installer.xml` — installer manifest for setting up an IRIS instance
 3. `SDAShredder-Export.xml` — all the classes required for an SDAShredder production
 4. `ISCPATIENTtoFHIR.xsd` — schema for an XML file
 5. `HS.SDA3.xsd` — schema for a CDA file

### Sample files
 1. `ISC/ISCPATIENTtoFHIR.xml` — sample XML file
 2. `ISC/EpicCCDA21toFHIR.xml` — sample CDA file
 3. `ISC/HL7toFHIR.HL7` — sample HL7 file
 4. `ISC/PatientsCSVtoFHIR.csv` — sample CSV file
 5. `ISC/In` — folder for dropping test files
 6. `README.md` — instructions to set up the code sample

## Installation instructions: 

1. Clone this repository: `git clone`.

2. Open the Terminal or command prompt window and change the directory to `SUREFHIRDemo`.

3. Execute the Docker Build command by doing one of the following:

* Execute the `build.sh` shell script.
* Use the Docker Build command: `docker build -t irishealth-surefhir-demo:1.0 .`

**Note:** the period at the end of the Docker Build command is **required**.

4. Execute the Docker Run command by doing one of the following:

* Execute  the `run.sh` shell script.
* Use the Docker Run command: `docker run -d --hostname SUREFHIR -p 52773:52773 --init -v $PWD/ISC:/tmp/ISC --name SUREFHIR irishealth-surefhir-demo:1.0`.

## Important notes

* If you need to change the webserver port to avoid port conflict, the argument format is NEWPORT:CURRENTPORT. For example, to switch to port 51773, you would set the parameter to 51773:52733.
* The hostname flag sets the server name of the container instance to *sdashredder*, which matches how the FHIR components are installed. If you change this to something else, you will need to adjust it per the Post Installation Notes, specifically #1.
* The production makes use of file drops, so you must map a folder to the Docker container. For this example, I have mapped a local folder path using the *-v* flag: `/users/tmp/docker/sdashredder/ISC` to the container folder `/tmp/ISC`.

## Check installation work
To verify you have installed succesfully:

1. Check the file path referenced in the file services within the SDAShredder production.
     - Go to Interoperability > Configure > System Default Settings.
     - Update the inbound file service path if needed.
     - Click the System Default Setting **FilePath**, and confirm the Setting Value is the drive you mounted during the `Docker Run` execution.
     - Make sure to click **Save** once you have made your changes.

3. Make sure all folders and sub-folders you have mapped to the Docker container have the correct read and write privileges set; otherwise, the file adapters will not be able to delete your test files after processing them.

## Test the sample FHIRStarter

Take a look at the `FHIRStarter` sample, which contains the following files:

* `ISC/ISCPATIENTtoFHIR.xml` — sample XML file
* `ISC/EpicCCDA21toFHIR.xml` — sample CDA file
* `ISC/HL7toFHIR.HL7` — sample HL7 file to be converted into FHIR
* `ISC/PatientsCSVtoFHIR.csv` — sample CSV file

Drop one sample file into the mapped folder and see that it is picked up and processed. If it works, you should receive a successful FHIR response.
Note that the `PatientsCSVtoFHIR` sample file has 200 patients to convert into FHIR resources. This can take up to 60 seconds depending on the system resources available to the demonstration instance.

## Test the sample FHIRPlace

Look at FHIRPlace sample, which contains the following files:

* `ISC/HL7toFHIR.HL7` — sample HL7 file to be converted into FHIR
* `ISC/HL7v2_ORU_R01.hl7` — sample HL7 file
* `ISC/MRNFlatFile.txt` — sample flat file with single MRN

To test it:
1. Drop `HL7toFHIR.HL7` into the input folder to add a FHIR resource to the *EMREmulatorFHIRR4* FHIR endpoint. The Service Registry entry `EMREmulatorFHIRR4` points to the `/fhirstarter/r4` FHIR endpoint. If you have already dropped `HL7toFHIR.HL7` into the input folder for the FHIRStarter demonstration, skip this step.
2. Drop either `HL7v2_ORU_R01.hl7` or `MRNFlatFile.txt` into the input folder to trigger the external FHIR data collection based on a patient MRN.  
The .hl7 message includes an MRN value in the list of patient identifiers that gets extracted by the `HL7v2.MRNExtractor`. 
The .txt file is picked up by a simple Record Map definition in the demonstration.
Both of these MRN input values are handed off to the PatientRecordCollector BPL.

Next, do the following: 

1. Delete any existing resources in the local `/fhirplace/r4` endpoint for the MRN from the input file. This assures there is no accidental duplication of data inside the FHIR repository.
2. Call out to an external FHIR endpoint labeled `EMREmulatorFHIRR4`. In this demonstration, that Service Registry points to the FHIRStarter endpoint `/fhirstarter/r4`. There are two calls to this endpoint:
* First, ask the endpoint, "Do you have a patient that matches the input MRN?".  
* If a patient is found, the second call is a `GET $everything` against the FHIR Patient resource.
This returns a *searchset* bundle that needs to be flipped to be saved into the InterSystems IRIS for Health™ FHIR Repository.
3. Send the 'searchset' bundle to the BundleFlip BPL. This component makes two major changes to the bundle.
* Change the type of bundle from *searchset* to *transaction* and update the necessary subfields to reflect this change.
* Change all the URL references to resources from `/<RESOURCE>/<ID>` to UUID values. Note that the `/<RESOURCE>/<ID>` values in the bundle will reflect the external endpoint resource IDs and need to be replaced in InterSystems IRIS for Health 2020.4 and earlier. Changing these references to UUIDs ensures that the InterSystems IRIS for Health FHIR code maintains the bundle's integrity when it is submitted to the local FHIR endpoint.
4. Submit the bundle to the local FHIR endpoint, `/fhirplace/r4`.
 

## Stop the sample

After you are done with this sample, you can enter the following commands to stop any containers that may still be running and remove them:

```
docker-compose stop
docker-compose rm
```

This is particularly helpful if you have other demos running on the same machine.



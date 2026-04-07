# Contributing to temurin-cdxa

## Description

This repository is for the contribution of CycloneDX Attestation (CDXA) documents
along with an accompanying detached signature from signing your documents.

The CDXA document confirms the successful (100% identical) 3rd party Reproducible Verification
of an Eclipse Temurin JDK.

For details on the Reproducible Verification process and guidelines,
see: https://github.com/adoptium/temurin-build/wiki/Temurin-3rd-Party-Reproducible-Verification-Guides

The Reproducible Verification scripts outlined in the [guide](https://github.com/adoptium/temurin-build/wiki/Temurin-3rd-Party-Reproducible-Verification-Guides) will only report
"success" upon a "100% identical" comparison.

For guidance on reporting "Reproducible Verification" failures or non-100% identical issues, please
see the problem diagnosis and reporting guidance here: https://github.com/adoptium/temurin-build/wiki/Temurin-3rd-Party-Reproducible-Verification-Guides#reporting-reproducible-verification-problems-or-failures

## CDXA.xml generation

- The easiest way to generate a valid CDXA.xml is by using a Java client class which can be obtained
and compiled from https://github.com/adoptium/temurin-build/blob/master/cyclonedx-lib/src/temurin/sbom/TemurinGenCDXA.java.
Requires any suitable jdk-17+ JDK and "ant" to build.
```
git clone https://github.com/adoptium/temurin-build.git
cd temurin-build/cyclonedx-lib
ant build

# Example usage, will generate CDXA.xml with the correct filename in the build directory
java -cp "build/jar/temurin-gen-cdxa.jar:build/jar/cyclonedx-core-java.jar:build/jar/jackson-core.jar:build/jar/jackson-dataformat-xml.jar:build/jar/jackson-databind.jar:build/jar/jackson-annotations.jar:build/jar/json-schema-validator.jar:build/jar/commons-codec.jar:build/jar/commons-io.jar:build/jar/github-package-url.jar:build/jar/webpki.org-libext-1.00.jar:build/jar/temurin-sign-sbom.jar:build/jar/commons-collections4.jar:build/jar/commons-lang3.jar:build/jar/stax2-api.jar:build/jar/woodstox-core.jar" temurin.sbom.TemurinGenCDXA \
--verbose \
--createNewCDXA \
--cdxa-output-folder build \
--attesting-org-name 'Acme Inc' \
--predicate VERIFIED_REPRODUCIBLE_BUILD \
--target-aarch x64 \
--target-os linux \
--target-release jdk-21.0.5+11 \
--verified-jdk-file /path-to-verified-tar.gz-or-zip \
--affirmation-stmt 'Acme confirms a verified reproducible build' \
--evidence 'Console log output from diff script......100% identical'

Output:
Computed SHA-256: 3c654d98404c073b8a7e66bffb27f4ae3e7ede47d13284c132d40a83144bfd8c
CDXA file written to: build/jdk_21_0_5_11_x64_linux_AcmeInc.xml
```

## CDXA.xml format

The CycloneDX Attestation document conforms to the schema defined here: https://cyclonedx.org/docs/1.6/json/#declarations_attestations

The specific Temurin CDXA.xml required format and validation rules, for this project
are as follows:
- The filename and path for the CDXA.xml must be :
  - \<major version number\>/\<jdk-tag\>/\<jdk-tag, -,.,+ changed to \_\>\_\<arch\>\_\<os\>\_\<your organization name, with no whitespace\>.xml
    - Example: 26/jdk-26+35/jdk_26_35_aarch64_linux_AcmeInc.xml
  - arch: x64, ppc64le, s390x, aarch64
  - os: linux, windows, mac
- The CDXA.xml must have been signed using a detached signature method, and submitted with the same name but with .xml.sig extension
  - Example: 26/jdk-26+35/jdk_26_35_aarch64_linux_AcmeInc.xml.sig

- Mandatory elements of the attestation declarations :
  - "attestation"
    - "summary" -> "Eclipse Temurin CycloneDX Attestation"
    - "assessor" and "claim" ref
  - "assessor"
    - "thirdParty" -> true
    - "organization" -> "your organization name"
  - "claim"
    - Must only be 1 "claim" under "claims"
    - "target" ref to the target "component" definition
    - "predicate" most be "VERIFIED_REPRODUCIBLE_BUILD"
    - "evidence" ref to the "evidence" definition
  - "evidence"
    - Must only be 1 "evidence" under top-level "evidence"
    - "propertyName" must be "VERIFICATION_LOG"
    - "data"
      - "name" must be "log"
      - "contents" must provide a "text/plain" "attachment" free form text, which contains the output from the Reproducible Verification process confirming 100% identical
  - "targets"
    - Must only contain 1 "component" with type "application"
    - "name" must be a correctly formatted string of the form: "Temurin TAG ARCH_OS", eg. "Temurin jdk-21.0.5+11 x64_linux"
      - The ARCH and OS must be valid values as would be used when querying the Adoptium API for binaries, eg: https://api.adoptium.net/q/swagger-ui/#/Binary/getBinary
        - ARCH: x64, ppc64le, s390x, aarch64
        - OS: linux, windows, mac
    - "version" must contain the JDK tag, and must match what is put in the filename and path
    - "externalReferences" - "reference" with type="distribution"
      - "url" - Adoptium API to retrieve the exact binary that was verified:
        - https://api.adoptium.net/v3/binary/version/<TAG>/<OS>/<ARCH>/jdk/hotspot/normal/eclipse
      - "hash" - SHA-256 hash of the verified JDK tarball/zip matching the hash from the Adoptium API
    - "properties" :
      - "platform" -> "\<ARCH\>\_\<OS\>"
      - "imageType" -> "jdk"
      - "jvmImpl" -> "hotspot"
  - "affirmation"
    - "statement" - Affirmation containing free format word statement stating that you verified the reproducibility. Also this statement can be used to give guidance on how the CDXA.xml and CDXA.xml.sig signing can be verified from your organization public key.

## A full example CDXA.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<bom serialNumber="urn:uuid:30b383c7-01f5-45bf-b153-d1507eccfc80" version="1" xmlns="http://cyclonedx.org/schema/bom/1.6">
  <declarations>
    <assessors>
      <assessor bom-ref="assessor-1">
        <thirdParty>true</thirdParty>
        <organization>
          <name>Acme Inc</name>
        </organization>
      </assessor>
    </assessors>
    <attestations>
      <attestation>
        <summary>Eclipse Temurin CycloneDX Attestation</summary>
        <assessor>assessor-1</assessor>
        <map>
          <claims>
            <claim>claim-1</claim>
          </claims>
        </map>
      </attestation>
    </attestations>
    <claims>
      <claim bom-ref="claim-1">
        <target>target-jdk-1</target>
        <predicate>VERIFIED_REPRODUCIBLE_BUILD</predicate>
        <evidence>evidence-1</evidence>
      </claim>
    </claims>
    <evidence>
      <evidence bom-ref="evidence-1">
        <propertyName>VERIFICATION_LOG</propertyName>
        <data>
          <name>log</name>
          <contents>
            <attachment content-type="text/plain">
Comparing expanded JDKs from jdk-21.0.5+11 with reproJDK/jdk-21.0.5+11
diff complete - rc=0. Output written to file: reprotest.diff
Number of differences: 0
ReproduciblePercent = 100 %
Compare identical !
            </attachment>
          </contents>
        </data>
      </evidence>
    </evidence>
    <targets>
      <components>
        <component type="application" bom-ref="target-jdk-1">
          <name>Temurin jdk-21.0.5+11 x64_linux</name>
          <version>jdk-21.0.5+11</version>
          <externalReferences>
            <reference type="distribution">
              <url>https://api.adoptium.net/v3/binary/version/jdk-21.0.5+11/linux/x64/jdk/hotspot/normal/eclipse</url>
              <hashes>
                <hash alg="SHA-256">3c654d98404c073b8a7e66bffb27f4ae3e7ede47d13284c132d40a83144bfd8c</hash>
              </hashes>
            </reference>
          </externalReferences>
          <properties>
            <property name="platform">x64_linux</property>
            <property name="imageType">jdk</property>
            <property name="jvmImpl">hotspot</property>
          </properties>
        </component>
      </components>
    </targets>
    <affirmation>
      <statement>
Acme Inc confirms a verified reproducible build.
This document is signed by public GPG key Acme Inc key details.
      </statement>
    </affirmation>
  </declarations>
</bom>
```



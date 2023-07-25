protobuf {
    // Configure the protoc executable
    protoc {
        artifact = 'com.google.protobuf:protoc:3.0.0'
        //        artifact = 'com.google.protobuf:protoc:3.0.0'
        // for apple m1, please add protoc_platform=osx-x86_64 in $HOME/.gradle/gradle.properties
        // mac m1芯片需要设置为true，其他为false。否则不能正常使用protobuf
        def isMacM1 = true
        if (isMacM1) {
            artifact = "com.google.protobuf:protoc:3.0.0:osx-x86_64"
        } else {
            artifact = "com.google.protobuf:protoc:3.0.0"
        }
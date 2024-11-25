## First blog

Welcome to my blog! Here, you will find me posting about things that I care and want to write about (simple act called blogging!)

This blog is powered by [chadbaldwin's blog template](https://github.com/chadbaldwin/simple-blog-bootstrap) for now.

Here's a simple `bash` code snippet to test out `highlightjs`. Taken from [here](https://gist.github.com/QuanTrieuPCYT/b77301ce922fa091c8ff6f5d01940d71).
```bash
#!/usr/bin/env bash
#
# Script to build ExcellentEnchants 4.0.5 with removed MC 1.20.6 support and Java 17 support.
# Provided that your environment meets the followings:
# - JDK 17 or later installed
# - `git` installed, `mvn` installed (Apache Maven)
# - Using an OS that uses GNU's `sed` utility (such as Linux). macOS uses a different `sed`, so please double-check!
#

# Clone the repository
git clone https://github.com/nulli0n/ExcellentEnchants-spigot ./ExcellentEnchants-spigot
cd ./ExcellentEnchants-spigot
# Revert to the 4.0.5 commit
git reset --hard 359782ff57d10ed2433b983a98dfc9a5663336d4
# Delete MC 1.20.6's NMS module
rm -R MC_1_20_6
# Clone the nightcore repository
git clone https://github.com/nulli0n/nightcore-spigot ./nightcore-spigot
cd ./nightcore-spigot
# Revert to the commit that supports this release (Core 2.6.2 Beta)
git reset --hard 8823d0eecdb67e7991f7aa81e2784d80cbf40917
# Build and install nightcore to our local Maven repository
mvn install
cd ..
# Text replacing magics
# These commands will remove 1.20.6 NMS module support, and downgrade the Java build target requirement to 17
sed -i '/<module>MC_1_20_6<\/module>/d; s/21/17/g' pom.xml
sed -i '/<dependencies>/i <repositories>\n    <repository>\n        <id>nms-repo</id>\n        <url>https://repo.codemc.org/repository/nms/</url>\n    </repository>\n</repositories>' pom.xml
sed -i 's/<version>2.6.1<\/version>/<version>2.6.2<\/version>/' pom.xml
sed -i 's/21/17/g' ./API/pom.xml
sed -i 's/21/17/g' ./Core/pom.xml
sed -i 's/<version>5.6.0-SNAPSHOT<\/version>/<version>5.6.2<\/version>/' ./Core/pom.xml
sed -i 's#<id>lumine-snapshot</id>#<id>papermc</id>#; s#<url>http://mvn.lumine.io/repository/maven-snapshots/</url>#<url>https://repo.papermc.io/repository/maven-public/</url>#' ./Core/pom.xml
sed -i -e '/<artifactId>MC_1_20_6<\/artifactId>/ { N; N; N; N; d; }' ./Core/pom.xml
sed -i '/MC_1_20_6/d' ./Core/src/main/java/su/nightexpress/excellentenchants/EnchantsPlugin.java
sed -i 's/21/17/g' ./NMS/pom.xml
sed -i 's/21/17/g' ./V1_19_R3/pom.xml
sed -i 's/21/17/g' ./V1_20_R1/pom.xml
sed -i 's/21/17/g' ./V1_20_R2/pom.xml
sed -i 's/21/17/g' ./V1_20_R3/pom.xml
# Actually build the plugin
mvn clean package
# The JAR will be in the folder `./target/`
```
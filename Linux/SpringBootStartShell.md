##### start_bases.sh

```
#!/bin/bash
BIN_DIR=`pwd`
cd ..
DEPLOY_DIR=`pwd`
PROP_DIR=$DEPLOY_DIR/bin/properties
source $PROP_DIR/debug.properties
source $PROP_DIR/jvm.properties

SERVER_NAME=servername

#LOG LOCATION
LOGS_DIR=$DEPLOY_DIR/logs
if [ ! -d $LOGS_DIR ]; then
    mkdir $LOGS_DIR
fi

PIDS=`ps -ef | grep java | grep "$DEPLOY_DIR" |awk '{print $2}'`
if [ -n "$PIDS" ]; then
    echo "ERROR: The $SERVER_NAME already started!"
    echo "PID: $PIDS"
    exit 1
fi

CONSOLE_TXT="Starting the $SERVER_NAME"

LIB_DIR=$DEPLOY_DIR/lib
APP_LIB_DIR=$DEPLOY_DIR/app
LIB_JARS=$LIB_DIR"/*"
APP_LIB_JARS=$APP_LIB_DIR"/*"
APP_JAR=$DEPLOY_DIR"/*.jar"
#Should put LIB_JARS ahead of the APP_LIB_JARS, so that the app folder's jar files won't override the lib folder's jar files.
JAVA_OPTS=" -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true "
if [ "true" = "$LOG_GC" ]; then
    JAVA_OPTS=$JAVA_OPTS" -verbose:gc -Xloggc:logs/gc.log -XX:+PrintGCDetails "
fi
JAVA_DEBUG_OPTS=""
if [ "$1" = "debug" ]; then
    JAVA_DEBUG_OPTS=" -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=${DEBUG_PORT},server=y,suspend=n "
    CONSOLE_TXT="$CONSOLE_TXT in DEBUG mode, the debug port is ${DEBUG_PORT}."
fi
JAVA_JMX_OPTS=""
if [ "$1" = "jmx" ]; then
    JAVA_JMX_OPTS=" -Dcom.sun.management.config.file=$PROP_DIR/jmx.properties "
    JMX_PORT=`awk -F= '/com.sun.management.jmxremote.port=/ {print $2}' bin/properties/jmx.properties`
    CONSOLE_TXT="$CONSOLE_TXT in JMX mode, the default jmx port is ${JMX_PORT}."
fi
JAVA_MEM_OPTS=" -server -Xmx${HEAP_MAX} -Xms${HEAP_INIT} -Xmn${YOUNG_GEN_SIZE} -XX:PermSize=${PERM_SIZE} -Xss${STACK_SIZE} -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:LargePageSizeInBytes=${LARGE_PAGE_SIZE_IN_BYTES} -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 "

echo $CONSOLE_TXT
```
##### start.sh

```
#!/bin/bash
cd `dirname $0`
source start_base
#make sure classpath .. is ahead of the lib path
nohup java $JAVA_OPTS $JAVA_MEM_OPTS $JAVA_DEBUG_OPTS $JAVA_JMX_OPTS -jar $APP_JAR > /dev/null 2>&1 &
echo "Please check the log file in $LOGS_DIR."
```
##### stop.sh


```
#!/bin/bash
cd `dirname $0`
BIN_DIR=`pwd`
cd ..
DEPLOY_DIR=`pwd`

SERVER_NAME=dd-frame-container

PIDS=`ps -ef | grep java | grep "$DEPLOY_DIR" |awk '{print $2}'`
if [ -z "$PIDS" ]; then
    echo "ERROR: The $SERVER_NAME does not started!"
    exit 1
fi

if [ "$1" = "dump" ]; then
    $BIN_DIR/dump.sh
fi

echo -e "Stopping the $SERVER_NAME ...\c"
for PID in $PIDS ; do
    kill $PID > /dev/null 2>&1
done

COUNT=0
while [ $COUNT -lt 1 ]; do
    echo -e ".\c"
    sleep 1
    COUNT=1
    for PID in $PIDS ; do
        PID_EXIST=`ps -f -p $PID | grep java`
        if [ -n "$PID_EXIST" ]; then
            COUNT=0
            break
        fi
    done
done

echo "OK!"
echo "PID: $PIDS"
```
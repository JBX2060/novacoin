cmake_minimum_required(VERSION 3.5)

##
## mkdir build && cd build
##
## cmake -DBerkeleyDB_ROOT:STRING=/opt/homebrew/Cellar/berkeley-db@4/4.8.30 -DQt5_DIR:STRING=/opt/homebrew/Cellar/qt@5/5.15.2_1/lib/cmake/Qt5 ..
## cmake ..
##

project(novacoin-qt VERSION 0.5.9 LANGUAGES C CXX)

# Enforce sizeof(size_t) == 8 by default
if (NOT ALLOW_32BIT AND NOT CMAKE_SIZEOF_VOID_P MATCHES "8")
    message(FATAL_ERROR "Only 64-bit processors (x86_64, AArch64) are supported")
endif ()

# Force generic scrypt on 32 bit platforms
if (NOT CMAKE_SIZEOF_VOID_P MATCHES "8")
    set(USE_GENERIC_SCRYPT True)
endif()

if (APPLE)
enable_language(OBJCXX)
endif()

find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
    set(CMAKE_C_COMPILER_LAUNCHER ccache)
    set(CMAKE_CXX_COMPILER_LAUNCHER ccache)
    if (APPLE)
    set(CMAKE_OBJCXX_COMPILER_LAUNCHER ccache)
    endif()
endif(CCACHE_FOUND)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(Qt5 COMPONENTS LinguistTools Core Widgets Gui Network Qml REQUIRED)

if (NOT MSVC)
if (NOT WIN32)
set(FPIC "-fPIC")
endif()
set (CMAKE_C_FLAGS "-flto=auto -O3 ${FPIC} -Wno-deprecated -Wno-deprecated-declarations -fstack-protector-all")
set (CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-parameter -fexceptions -frtti")
else()
set(CMAKE_C_FLAGS "/O2")
set(CMAKE_CXX_FLAGS "/O2 /GR /EHsc")
endif()

set(Boost_USE_STATIC_LIBS   ON)
if(WIN32)
set(Boost_USE_STATIC_RUNTIME ON)
endif()
find_package(Boost REQUIRED COMPONENTS atomic chrono filesystem program_options system thread)

set(OPENSSL_USE_STATIC_LIBS ON)
find_package(OpenSSL)
if (NOT OpenSSL_FOUND)
set(OPENSSL_USE_STATIC_LIBS OFF)
find_package(OpenSSL REQUIRED)
endif()

# Disable compiling qrencode executables
option(WITH_TOOLS "Build utility tools"  NO)
add_subdirectory(src/additional/libqrencode)
add_subdirectory(src/additional/IXWebSocket)

# Set default root path for berkeley db
if (NOT BerkeleyDB_ROOT)
set(BerkeleyDB_ROOT /usr)
endif()

# Set default include path for berkeley db
if (NOT BerkeleyDB_INC)
set(BerkeleyDB_INC ${BerkeleyDB_ROOT}/include)
endif()

if (NOT EXISTS ${BerkeleyDB_INC}/db_cxx.h)
    message(FATAL_ERROR "Unable to find db_cxx.h header file in ${BerkeleyDB_INC} directory. Try to specify either include directory path via BerkeleyDB_INC or berkeley db root path via BerkeleyDB_ROOT variable.")
endif()

# Set default library path for berkeley db
if (NOT BerkeleyDB_LIBS)
set(BerkeleyDB_LIBS ${BerkeleyDB_ROOT}/lib)
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
# Workaround for linux distributions
    execute_process (
        COMMAND bash -c "test -d ${BerkeleyDB_LIBS}/${CMAKE_SYSTEM_PROCESSOR}-linux-gnu && echo -n 'present' || echo -n 'absent'"
        OUTPUT_VARIABLE dirPresent
    )
    if(dirPresent STREQUAL "present")
        set(BerkeleyDB_LIBS ${BerkeleyDB_LIBS}/${CMAKE_SYSTEM_PROCESSOR}-linux-gnu)
    endif()
endif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
endif()

# Set default library name for berkeley db
if (NOT BerkeleyDB_LIB_name)
set(BerkeleyDB_LIB_name libdb_cxx)
endif()

set(generic_sources 
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/bitcoin.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/bitcoingui.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/intro.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/transactiontablemodel.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/addresstablemodel.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/optionsdialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/sendcoinsdialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/coincontroldialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/coincontroltreewidget.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/addressbookpage.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/signverifymessagedialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/aboutdialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/editaddressdialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/bitcoinaddressvalidator.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/trafficgraphwidget.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/mintingfilterproxy.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/mintingtablemodel.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/mintingview.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/allocators.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/bignum.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/coincontrol.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/kernelrecord.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/alert.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/version.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/sync.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/util.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/timedata.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/netbase.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ntp.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/key.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/script.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/streams.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/miner.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/random.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/init.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/net.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/stun.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/irc.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/checkpoints.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/addrman.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/db.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/walletdb.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/uint256.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/clientmodel.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/guiutil.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/transactionrecord.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/optionsmodel.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/monitoreddatamapper.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/transactiondesc.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/transactiondescdialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/bitcoinstrings.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/bitcoinamountfield.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/wallet.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/keystore.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/transactionfilterproxy.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/transactionview.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/walletmodel.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/bitcoinrpc.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/rpcdump.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/rpcnet.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/rpcmining.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/rpcwallet.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/rpcblockchain.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/rpcrawtransaction.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/overviewpage.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/csvmodelwriter.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/crypter.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/sendcoinsentry.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/qvalidatedlineedit.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/bitcoinunits.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/qvaluecombobox.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/askpassphrasedialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/protocol.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/notificator.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/qtipcserver.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/rpcconsole.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/noui.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/kernel.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/kernel_worker.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/multisigaddressentry.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/multisiginputentry.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/multisigdialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/secondauthdialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/qrcodedialog.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/base58.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ipcollector.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/intro.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/coincontroldialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/sendcoinsdialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/addressbookpage.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/signverifymessagedialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/aboutdialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/editaddressdialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/transactiondescdialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/overviewpage.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/sendcoinsentry.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/askpassphrasedialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/rpcconsole.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/optionsdialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/multisigaddressentry.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/multisiginputentry.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/multisigdialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/secondauthdialog.ui
    ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/qrcodedialog.ui
)

# Copy translations as a workaround for `make clean`
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/qt/locale/bitcoin_en.ts ${CMAKE_BINARY_DIR}/bitcoin_en.ts COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/qt/locale/bitcoin_ru.ts ${CMAKE_BINARY_DIR}/bitcoin_ru.ts COPYONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/qt/locale/bitcoin_uk.ts ${CMAKE_BINARY_DIR}/bitcoin_uk.ts COPYONLY)

# Create translations to be used in bitcoin.qrc
qt5_add_translation(bitcoin_en_qm ${CMAKE_BINARY_DIR}/bitcoin_en.ts)
qt5_add_translation(bitcoin_ru_qm ${CMAKE_BINARY_DIR}/bitcoin_ru.ts)
qt5_add_translation(bitcoin_uk_qm ${CMAKE_BINARY_DIR}/bitcoin_uk.ts)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/qt/bitcoin.qrc ${CMAKE_BINARY_DIR}/bitcoin.qrc)

list(APPEND ALL_SOURCES ${generic_sources} ${CMAKE_BINARY_DIR}/bitcoin.qrc)
list(APPEND ALL_LIBRARIES ${Boost_LIBRARIES} ixwebsocket OpenSSL::Crypto qrencode Qt5::Gui Qt5::Widgets Qt5::Core Qt5::Network)

if (WIN32)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/qt/res/bitcoin-qt.rc ${CMAKE_BINARY_DIR}/bitcoin-qt.rc)
    list(APPEND ALL_SOURCES ${CMAKE_BINARY_DIR}/bitcoin-qt.rc)
    list(APPEND ALL_LIBRARIES ws2_32 shlwapi mswsock ole32 oleaut32 uuid gdi32 pthread)
    list(APPEND ALL_DEFINITIONS WIN32)
endif()

if (APPLE)
    list(APPEND ALL_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/res/icons/bitcoin.icns ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/macdockiconhandler.mm ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/macnotificationhandler.mm)
    list(APPEND ALL_LIBRARIES "-framework Foundation -framework ApplicationServices -framework AppKit")
endif()

# Some systems need rt library
if (NOT WIN32 AND NOT APPLE)
    list(APPEND ALL_LIBRARIES rt)
endif()

# Try various libdb library file extensions
foreach(ext a so dylib lib)
if (EXISTS ${BerkeleyDB_LIBS}/${BerkeleyDB_LIB_name}.${ext})
    list(APPEND ALL_LIBRARIES ${BerkeleyDB_LIBS}/${BerkeleyDB_LIB_name}.${ext})
    set (BDB_FOUND True)
    break()
endif()
endforeach()

if (NOT BDB_FOUND)
    message(FATAL_ERROR "Unable to find ${BerkeleyDB_LIB_name} library in ${BerkeleyDB_LIBS} directory. Try to specify either library path via BerkeleyDB_LIBS or berkeley db root path via BerkeleyDB_ROOT variable. You can also use BerkeleyDB_LIB_name variable to provide alternative file name search for.")
endif()

list(APPEND ALL_DEFINITIONS QT_GUI USE_IPV6 BOOST_SPIRIT_THREADSAFE)

# Disable useless targets
option(LEVELDB_BUILD_TESTS "Build LevelDB's unit tests" OFF)
option(LEVELDB_BUILD_BENCHMARKS "Build LevelDB's benchmarks" OFF)
option(LEVELDB_INSTALL "Install LevelDB's header and library" OFF)

add_subdirectory(src/additional/leveldb)
list(APPEND ALL_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/txdb-leveldb.cpp)
list(APPEND ALL_LIBRARIES leveldb)

if (NOT USE_GENERIC_SCRYPT)
    list( APPEND ALL_SOURCES ${generic_sources} ${CMAKE_CURRENT_SOURCE_DIR}/src/crypto/scrypt/intrin/scrypt-intrin.cpp )
    list(APPEND ALL_DEFINITIONS USE_INTRIN)
else()
    list( APPEND ALL_SOURCES ${generic_sources} ${CMAKE_CURRENT_SOURCE_DIR}/src/crypto/scrypt/generic/scrypt-generic.cpp )
endif()

# Generate build info header
execute_process (
    COMMAND sh -c "${CMAKE_CURRENT_SOURCE_DIR}/share/genbuild.sh ${CMAKE_CURRENT_SOURCE_DIR}/src/build.h"
    OUTPUT_VARIABLE outVar
)
list(APPEND ALL_DEFINITIONS HAVE_BUILD_INFO)

add_executable(novacoin-qt WIN32 ${ALL_SOURCES})

target_precompile_headers(novacoin-qt
  PRIVATE
    <algorithm>
    <cassert>
    <cerrno>
    <cinttypes>
    <climits>
    <condition_variable>
    <csignal>
    <cstdarg>
    <cstdint>
    <cstdio>
    <cstring>
    <ctime>
    <deque>
    <fstream>
    <functional>
    <iomanip>
    <ios>
    <iostream>
    <limits>
    <list>
    <map>
    <memory>
    <mutex>
    <ostream>
    <regex>
    <set>
    <stdexcept>
    <sstream>
    <string>
    <tuple>
    <type_traits>
    <variant>
    <vector>
    <utility>
    <QtAlgorithms>
    <QtGlobal>
    <QtPlugin>
    <QAbstractButton>
    <QAbstractItemDelegate>
    <QAbstractItemModel>
    <QAbstractItemView>
    <QAbstractListModel>
    <QAbstractTableModel>
    <QAction>
    <QApplication>
    <QByteArray>
    <QCheckBox>
    <QClipboard>
    <QColor>
    <QComboBox>
    <QCompleter>
    <QCursor>
    <QDataWidgetMapper>
    <QDateTime>
    <QDesktopServices>
    <QDialog>
    <QDialogButtonBox>
    <QDir>
    <QDoubleSpinBox>
    <QDoubleValidator>
    <QDragEnterEvent>
    <QEvent>
    <QEventLoop>
    <QFile>
    <QFileDialog>
    <QFlags>
    <QFont>
    <QFrame>
    <QHeaderView>
    <QIcon>
    <QImage>
    <QImageWriter>
    <QIntValidator>
    <QKeyEvent>
    <QLabel>
    <QLibraryInfo>
    <QLineEdit>
    <QList>
    <QLocale>
    <QMainWindow>
    <QMenu>
    <QMenuBar>
    <QMessageBox>
    <QMetaObject>
    <QMetaProperty>
    <QMetaType>
    <QMimeData>
    <QModelIndex>
    <QMovie>
    <QMutex>
    <QObject>
    <QPainter>
    <QPainterPath>
    <QPixmap>
    <QPoint>
    <QProgressBar>
    <QPushButton>
    <QQueue>
    <QRegExp>
    <QRegExpValidator>
    <QScrollBar>
    <QSet>
    <QSettings>
    <QSignalMapper>
    <QSortFilterProxyModel>
    <QSplashScreen>
    <QStackedWidget>
    <QStandardPaths>
    <QStatusBar>
    <QString>
    <QStringList>
    <QStyle>
    <QSystemTrayIcon>
    <QTableView>
    <QTabWidget>
    <QTemporaryFile>
    <QTextCodec>
    <QTextDocument>
    <QTextEdit>
    <QTextStream>
    <QThread>
    <QTime>
    <QTimer>
    <QToolBar>
    <QTranslator>
    <QTreeWidget>
    <QTreeWidgetItem>
    <QUrl>
    <QUrlQuery>
    <QWidget>
    <QVariant>
    <QVBoxLayout>
    <QHBoxLayout>
)

if (NOT MSVC)
list(APPEND ALL_DEFINITIONS _FORTIFY_SOURCE=2)
set(ALL_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fstack-protector-all")
if (WIN32)
list(APPEND ALL_LIBRARIES mingw32 ssp)
set(ALL_LINKER_FLAGS "${ALL_LINKER_FLAGS} -static-libgcc -static")
endif()
set(CMAKE_EXE_LINKER_FLAGS ${ALL_LINKER_FLAGS} CACHE INTERNAL "")
endif()

target_include_directories(novacoin-qt PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/src/qt ${CMAKE_CURRENT_SOURCE_DIR}/src/json ${BerkeleyDB_INC} ${CMAKE_CURRENT_SOURCE_DIR}/src/additional/leveldb/helpers ${CMAKE_CURRENT_SOURCE_DIR}/src/additional/libqrencode ${Boost_INCLUDE_DIRS})
target_link_libraries(novacoin-qt ${ALL_LIBRARIES})

target_compile_features(novacoin-qt PUBLIC cxx_std_17)

set_property(TARGET novacoin-qt PROPERTY CXX_STANDARD 17)
set_property(TARGET novacoin-qt PROPERTY CXX_STANDARD_REQUIRED TRUE)
set_property(TARGET novacoin-qt PROPERTY COMPILE_DEFINITIONS ${ALL_DEFINITIONS})
set_property(TARGET novacoin-qt PROPERTY CMAKE_WARN_DEPRECATED FALSE)

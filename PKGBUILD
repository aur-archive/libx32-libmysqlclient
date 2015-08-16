# $Id: PKGBUILD 167245 2012-09-29 09:31:54Z andrea $
# Maintainer: Andrea Scarpino <andrea@archlinux.org>
# Contributor: Douglas Soares de Andrade <douglas@archlinux.org>

_pkgbase=mysql
pkgbase=binx32-mysql
#pkgname=('libx32-libmysqlclient' 'binx32-mysql-clients' 'binx32-mysql')
pkgname=('libx32-libmysqlclient')
pkgver=5.5.28
pkgrel=1.1
arch=('x86_64')
license=('GPL')
url="https://www.mysql.com/products/community/"
makedepends=('cmake' 'libx32-openssl' 'libx32-zlib')
options=('!libtool')
source=("http://ftp.gwdg.de/pub/misc/mysql/Downloads/MySQL-5.5/${_pkgbase}-${pkgver}.tar.gz"
        'mysqld.rc' 'my.cnf' 'mysqld-post.sh' 'mysqld-tmpfile.conf' 'mysqld.service')
md5sums=('da8ddb3d00e6e159153d2444360f52a8'
         'e7eb2faeab755cbb2fe03542bf328da9'
         '1c949c0dbea5206af0db14942d9927b6'
         '6ea6ceb360d09a774e87335ee098ddd5'
         '2fa6e456964d4ff5e6d4f9ff0126aed6'
         'a0e1460c276a50a9666c331de83271f2')

build() {
  mkdir build
  cd build

  # CFLAGS/CXXFLAGS as suggested upstream

  cmake ../${_pkgbase}-${pkgver} \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DSYSCONFDIR=/etc/mysql \
    -DMYSQL_DATADIR=/var/lib/mysql \
    -DMYSQL_UNIX_ADDR=/run/mysqld/mysqld.sock \
    -DDEFAULT_CHARSET=utf8 \
    -DDEFAULT_COLLATION=utf8_general_ci \
    -DENABLED_LOCAL_INFILE=ON \
    -DINSTALL_LIBDIR=libx32 \
    -DINSTALL_INFODIR=share/mysql/docs \
    -DINSTALL_MANDIR=share/man \
    -DINSTALL_PLUGINDIR=lib/mysql/plugin \
    -DINSTALL_SCRIPTDIR=bin \
    -DINSTALL_INCLUDEDIR=include/mysql \
    -DINSTALL_DOCREADMEDIR=share/mysql \
    -DINSTALL_SUPPORTFILESDIR=share/mysql \
    -DINSTALL_MYSQLSHAREDIR=share/mysql \
    -DINSTALL_DOCDIR=share/mysql/docs \
    -DINSTALL_SHAREDIR=share/mysql \
    -DWITH_READLINE=ON \
    -DWITH_ZLIB=system \
    -DWITH_SSL=system \
    -DWITH_LIBWRAP=OFF \
    -DWITH_MYSQLD_LDFLAGS="${LDFLAGS}" \
    -DWITH_EXTRA_CHARSETS=complex \
    -DWITH_EMBEDDED_SERVER=ON \
    -DWITH_INNOBASE_STORAGE_ENGINE=1 \
    -DWITH_PARTITION_STORAGE_ENGINE=1 \
    -DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 \
    -DWITHOUT_ARCHIVE_STORAGE_ENGINE=1 \
    -DWITHOUT_BLACKHOLE_STORAGE_ENGINE=1 \
    -DWITHOUT_FEDERATED_STORAGE_ENGINE=1 \
    -DCMAKE_C_FLAGS="-mx32 -fPIC ${CFLAGS} -fno-strict-aliasing -DBIG_JOINS=1 -fomit-frame-pointer" \
    -DCMAKE_CXX_FLAGS="-mx32 -fPIC ${CXXFLAGS} -fno-strict-aliasing -DBIG_JOINS=1 -felide-constructors -fno-rtti"

  make
}

package_libx32-libmysqlclient(){
  pkgdesc="MySQL client libraries (x32 ABI)"
  depends=(libmysqlclient=${pkgver} 'libx32-openssl')
  
  cd build
  for dir in include libmysql libmysqld libservices; do
    make -C ${dir} DESTDIR="${pkgdir}" install
  done

  install -d "${pkgdir}"/usr/bin
  install -m755 scripts/mysql_config "${pkgdir}"/usr/bin/mysql-x32_config

  rm -rf "${pkgdir}/usr/include"
}

package_mysql-clients(){
  pkgdesc="MySQL client tools"
  depends=('libmysqlclient')
  
  cd build
  make -C client DESTDIR="${pkgdir}" install

  # install man pages
  install -d "${pkgdir}"/usr/share/man/man1
  for man in mysql mysqladmin mysqlcheck mysqldump mysqlimport mysqlshow mysqlslap; do
    install -m644 "${srcdir}"/${_pkgbase}-${pkgver}/man/$man.1 "${pkgdir}"/usr/share/man/man1/$man.1
  done

  # provided by mysql
  rm "${pkgdir}"/usr/bin/{mysql_{plugin,upgrade},mysqlbinlog,mysqltest}
}

package_mysql(){
  pkgdesc="A fast SQL database server"
  backup=('etc/mysql/my.cnf')
  install=mysql.install
  depends=('mysql-clients' 'systemd-tools')
  options=('emptydirs')

  cd build
  make DESTDIR="${pkgdir}" install

  install -Dm644 "${srcdir}"/my.cnf "${pkgdir}"/etc/mysql/my.cnf
  install -Dm755 "${srcdir}"/mysqld.rc "${pkgdir}"/etc/rc.d/mysqld
  install -Dm755 "${srcdir}"/mysqld-post.sh "${pkgdir}"/usr/bin/mysqld-post
  install -Dm644 "${srcdir}"/mysqld-tmpfile.conf "${pkgdir}"/usr/lib/tmpfiles.d/mysqld.conf
  install -d "${pkgdir}"/usr/lib/systemd/system
  install -Dm644 "${srcdir}"/mysqld.service "${pkgdir}"/usr/lib/systemd/system/

  # provided by libmysqlclient
  rm "${pkgdir}"/usr/bin/{mysql_config,mysql_client_test_embedded,mysqltest_embedded}
  rm "${pkgdir}"/usr/lib/libmysql*
  rm -r "${pkgdir}"/usr/include/
  rm "${pkgdir}"/usr/share/man/man1/{mysql_config,mysql_client_test_embedded,mysqltest_embedded}.1
  
  # provided by mysql-clients
  rm "${pkgdir}"/usr/bin/{mysql,mysqladmin,mysqlcheck,mysqldump,mysqlimport,mysqlshow,mysqlslap}
  rm "${pkgdir}"/usr/share/man/man1/{mysql,mysqladmin,mysqlcheck,mysqldump,mysqlimport,mysqlshow,mysqlslap}.1

  # not needed
  rm -r "${pkgdir}"/usr/{data,mysql-test,sql-bench}
  rm "${pkgdir}"/usr/share/man/man1/mysql-test-run.pl.1

  install -dm700 "${pkgdir}"/var/lib/mysql
}

# websocketpp bug reproduce

Commit https://github.com/bincrafters/conan-websocketpp/commit/212cd56c856ca0c64be28da89d7c82d2e74fe07b bumped the requirement from OpenSSL/1.1.1c to openssl/1.1.1d

If you also require OpenSSL/1.1.1c in your workflow, there will be an error. This is a MCVE to reproduce the error.

```
-- Conan: Loading conanbuildinfo.cmake
-- Conan: Using cmake targets configuration
-- Library ssl found /home/julien/.conan/data/openssl/1.1.1d/_/_/package/b781af3f476d0aa5070a0a35b544db7a3c193cc8/lib/libssl.a
-- Library crypto found /home/julien/.conan/data/openssl/1.1.1d/_/_/package/b781af3f476d0aa5070a0a35b544db7a3c193cc8/lib/libcrypto.a
-- Library dl not found in package, might be system one
-- Library pthread not found in package, might be system one
-- Library ssl found /home/julien/.conan/data/openssl/1.1.1d/_/_/package/b781af3f476d0aa5070a0a35b544db7a3c193cc8/lib/libssl.a
CMake Error at build/conanbuildinfo.cmake:726 (add_library):
  add_library cannot create imported target "CONAN_LIB::OpenSSL_ssl" because
  another target with the same name already exists.
Call Stack (most recent call first):
  build/conanbuildinfo.cmake:414 (conan_package_library_targets)
  build/conanbuildinfo.cmake:608 (conan_define_targets)
  build/conan.cmake:503 (conan_basic_setup)
  CMakeLists.txt:75 (conan_cmake_run)


-- Library crypto found /home/julien/.conan/data/openssl/1.1.1d/_/_/package/b781af3f476d0aa5070a0a35b544db7a3c193cc8/lib/libcrypto.a
CMake Error at build/conanbuildinfo.cmake:726 (add_library):
  add_library cannot create imported target "CONAN_LIB::OpenSSL_crypto"
  because another target with the same name already exists.
Call Stack (most recent call first):
  build/conanbuildinfo.cmake:414 (conan_package_library_targets)
  build/conanbuildinfo.cmake:608 (conan_define_targets)
  build/conan.cmake:503 (conan_basic_setup)
  CMakeLists.txt:75 (conan_cmake_run)


-- Library dl not found in package, might be system one
-- Library pthread not found in package, might be system one
CMake Error at build/conanbuildinfo.cmake:430 (add_library):
  add_library cannot create imported target "CONAN_PKG::OpenSSL" because
  another target with the same name already exists.
Call Stack (most recent call first):
  build/conanbuildinfo.cmake:608 (conan_define_targets)
  build/conan.cmake:503 (conan_basic_setup)
  CMakeLists.txt:75 (conan_cmake_run)

```

#==================================================================================================#
#                                                                                                  #
#  Copyright (c) 2012 MaidSafe.net limited                                                         #
#                                                                                                  #
#  The following source code is property of MaidSafe.net limited and is not meant for external     #
#  use.  The use of this code is governed by the license file licence.txt found in the root        #
#  directory of this project and also on www.maidsafe.net.                                         #
#                                                                                                  #
#  You are not free to copy, amend or otherwise use this source code without the explicit written  #
#  permission of the board of directors of MaidSafe.net.                                           #
#                                                                                                  #
#==================================================================================================#


ms_get_branch(Branch)
set(Msg "Configuring third party projects on ${Branch} branch of super-project")
string(REGEX REPLACE . "-" Underscore ${Msg})
message("${HR}\n${Msg}\n${Underscore}")


include(add_boost)
add_subdirectory(catch/projects/CMake)
add_subdirectory(cryptopp)
add_subdirectory(googlemock)
add_subdirectory(protobuf)
add_subdirectory(leveldb)


# Add third party tests
if(MaidsafeTesting)
  set(CamelCaseProjectName ThirdParty)
  set(AllExesForCurrentProject cryptest ${AllLeveldbTests} leveldb_benchmark)
  ms_add_project_experimental()
  add_custom_command(TARGET AllThirdParty
                       PRE_BUILD
                       COMMAND ${CMAKE_COMMAND} ARGS -E copy_directory $<TARGET_FILE_DIR:cryptest>/TestData ${CMAKE_CURRENT_BINARY_DIR}/TestData)
  add_custom_command(TARGET AllThirdParty
                       PRE_BUILD
                       COMMAND ${CMAKE_COMMAND} ARGS -E copy_directory $<TARGET_FILE_DIR:cryptest>/TestVectors ${CMAKE_CURRENT_BINARY_DIR}/TestVectors)
  add_test(NAME "\"CryptoPP full validataion\"" COMMAND cryptest v)
  set_tests_properties("\"CryptoPP full validataion\"" PROPERTIES TIMEOUT 60 LABELS ThirdParty Behavioural CryptoPP)
  ms_add_memcheck_ignore("CryptoPP full validataion")
  add_test(NAME "\"CryptoPP test vectors for AES\"" COMMAND cryptest tv aes)
  set_tests_properties("\"CryptoPP test vectors for AES\"" PROPERTIES TIMEOUT 60 LABELS ThirdParty Behavioural CryptoPP)
  ms_add_memcheck_ignore("CryptoPP test vectors for AES")
  add_test(NAME "\"CryptoPP test vectors for SHA\"" COMMAND cryptest tv sha)
  set_tests_properties("\"CryptoPP test vectors for SHA\"" PROPERTIES TIMEOUT 60 LABELS ThirdParty Behavioural CryptoPP)
  ms_add_memcheck_ignore("CryptoPP test vectors for SHA")
  add_test(NAME "\"CryptoPP test vectors for RSA-PSS\"" COMMAND cryptest tv rsa_pss)
  set_tests_properties("\"CryptoPP test vectors for RSA-PSS\"" PROPERTIES TIMEOUT 60 LABELS ThirdParty Behavioural CryptoPP)
  ms_add_memcheck_ignore("CryptoPP test vectors for RSA-PSS")
  add_test(NAME "\"CryptoPP test vectors for RSA-OAEP\"" COMMAND cryptest tv rsa_oaep)
  set_tests_properties("\"CryptoPP test vectors for RSA-OAEP\"" PROPERTIES TIMEOUT 60 LABELS ThirdParty Behavioural CryptoPP)
  ms_add_memcheck_ignore("CryptoPP test vectors for RSA-OAEP")
  add_test(NAME "\"CryptoPP test vectors for RSA-PKCS1 v1.5\"" COMMAND cryptest tv rsa_pkcs1_1_5)
  set_tests_properties("\"CryptoPP test vectors for RSA-PKCS1 v1.5\"" PROPERTIES TIMEOUT 60 LABELS ThirdParty Behavioural CryptoPP)
  ms_add_memcheck_ignore("CryptoPP test vectors for RSA-PKCS1 v1.5")

  foreach(LeveldbTest ${AllLeveldbTests})
    string(REPLACE "test_leveldb_" "LevelDB " TestName ${LeveldbTest})
    add_test(NAME "\"${TestName}\"" COMMAND ${LeveldbTest})
    set_tests_properties("\"${TestName}\"" PROPERTIES TIMEOUT 60 LABELS ThirdParty Behavioural LevelDB)
    ms_add_memcheck_ignore("${TestName}")
  endforeach()
  set_tests_properties("\"LevelDB db\"" PROPERTIES TIMEOUT 900)
  set_tests_properties("\"LevelDB autocompact\"" "\"LevelDB issue178\"" "\"LevelDB table\"" PROPERTIES TIMEOUT 300)
endif()

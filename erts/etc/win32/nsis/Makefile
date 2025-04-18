# 
# %CopyrightBegin%
# 
# Copyright Ericsson AB 2003-2025. All Rights Reserved.
# 
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# 
# %CopyrightEnd%
# 

include $(ERL_TOP)/make/target.mk
include $(ERL_TOP)/make/$(TARGET)/otp.mk
include $(ERL_TOP)/erts/vsn.mk

VERSION_HEADER = erlang.nsh
MAKENSIS = makensis.exe
MAKENSISFLAGS = /V2
CUSTOM_MODERN=custom_modern.exe

# This is not the way we usually do in our makefiles,
# but making release is the ONLY thing we do with this one,
# Its not called during ordinary recursive make.
all: release

opt debug depend:
	@echo Nothing to do for "'"$@"'" on $(TARGET)

clean:
	rm -f $(VERSION_HEADER)

include $(ERL_TOP)/make/otp_release_targets.mk

TARGET_DIR = $(RELEASE_PATH)

ifdef MSYSTEM
 ifeq ($(MSYSTEM),$(filter $(MSYSTEM),MSYS MINGW32 MINGW64))
  MAKENSISFLAGS = //V2
 endif
endif

WTESTROOT=$(shell (w32_path.sh -d "$(RELEASE_PATH)"))
WTARGET_DIR=$(shell (w32_path.sh -d "$(TARGET_DIR)"))

ifeq ($(CONFIG_SUBTYPE),win64)
  WINTYPE=win64
  REDIST_TARGET=vcredist_x64.exe
else ifeq ($(CONFIG_SUBTYPE),arm64)
  WINTYPE=arm64
  REDIST_TARGET=vcredist_arm64.exe
else
  WINTYPE=win32
  REDIST_TARGET=vcredist_x86.exe
endif

REDIST_FILE=$(shell (sh ./find_redist.sh $(WINTYPE) || echo ""))
NICEREDISTFILE=$(shell (w32_path.sh -m "$(REDIST_FILE)" 2>/dev/null || echo "NOTFOUND"))
# $(info $$NICEREDISTFILE = [${NICEREDISTFILE}])
REDIST_DLL_VERSION=$(shell (sh ./dll_version_helper.sh "$(NICEREDISTFILE)" || echo ""))
REDIST_DLL_NAME=$(shell (sh ./dll_version_helper.sh -n "$(NICEREDISTFILE)" || echo ""))
# $(info $$REDIST_DLL_VERSION = [${REDIST_DLL_VERSION}])
# $(info $$REDIST_DLL_NAME = [${REDIST_DLL_NAME}])
# $(info $$REDIST_FILE = [${REDIST_FILE}])

VERSION_TICKETS_FILE=$(shell (w32_path.sh -m "$(ERL_TOP)/make/otp_version_tickets" 2>/dev/null || echo "NOTFOUND"))
# NSIS really want the product version number to be at least 4 digits long
OTP_VERSION_LONG=$(shell $(ERL_TOP)/bootstrap/bin/escript.exe vsn_number.escript $(OTP_VERSION) $(VERSION_TICKETS_FILE))
YEAR=$(shell date +%Y)

release_spec:
	@NSIS_VER=`makensis.exe -version`; \
	case $$NSIS_VER in \
	v2.* | v3.0 | v3.01) \
	  echo "Unsupported NSIS version: $$NSIS_VER";;\
	v3.*) \
	  echo '!define OTP_RELEASE "$(SYSTEM_VSN)"' > $(VERSION_HEADER);\
	  echo '!define OTP_VERSION "$(OTP_VERSION)"' >> $(VERSION_HEADER);\
	  echo '!define OTP_VERSION_LONG "$(OTP_VERSION_LONG)"' >> $(VERSION_HEADER);\
	  echo '!define ERTS_VERSION "$(VSN)"' >> $(VERSION_HEADER);\
	  echo '!define TESTROOT "$(WTESTROOT)"' >> $(VERSION_HEADER);\
	  echo '!define OUTFILEDIR "$(WTARGET_DIR)"' >> $(VERSION_HEADER);\
	  echo '!define WINTYPE "$(WINTYPE)"' >> $(VERSION_HEADER);\
	  echo '!define YEAR "$(YEAR)"' >> $(VERSION_HEADER);\
	  if [ -f $(CUSTOM_MODERN) ];\
	  then \
		echo '!define HAVE_CUSTOM_MODERN 1' >> $(VERSION_HEADER); \
	  fi;\
	  if [ '!' -z "$(REDIST_FILE)" -a '!' -z "$(REDIST_DLL_VERSION)" ];\
	  then \
		cp "$(REDIST_FILE)" "$(RELEASE_PATH)/$(REDIST_TARGET)";\
		echo '!define HAVE_REDIST_FILE 1'  >> $(VERSION_HEADER); \
		echo '!define REDIST_DLL_VERSION "$(REDIST_DLL_VERSION)"' >> $(VERSION_HEADER);\
		echo '!define REDIST_DLL_NAME "$(REDIST_DLL_NAME)"' >> $(VERSION_HEADER);\
		echo '!define REDIST_EXECUTABLE "$(REDIST_TARGET)"' >> $(VERSION_HEADER);\
	  fi;\
	  if [ -f "$(RELEASE_PATH)/docs/doc/index.html" ];\
	  then \
		echo '!define HAVE_DOCS 1' >> $(VERSION_HEADER); \
	  fi;\
	  echo "OTP_VERSION = [${OTP_VERSION}]"; \
	  echo "OTP_VERSION_LONG = [${OTP_VERSION_LONG}]"; \
	  echo "Running $(MAKENSIS) $(MAKENSISFLAGS) erlang20.nsi";\
	  $(MAKENSIS) $(MAKENSISFLAGS) erlang20.nsi;;\
	*) \
	  echo "Unsupported NSIS version: $$NSIS_VER";;\
	esac	

release_docs release_docs_spec docs:


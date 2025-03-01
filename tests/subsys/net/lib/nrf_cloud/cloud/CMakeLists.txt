#
# Copyright (c) 2022 Nordic Semiconductor
#
# SPDX-License-Identifier: LicenseRef-Nordic-5-Clause
#

cmake_minimum_required(VERSION 3.20.0)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(nrf_cloud_init_uninit_test)
set(NRF_SDK_DIR ${ZEPHYR_BASE}/../nrf)
cmake_path(NORMAL_PATH NRF_SDK_DIR)

# NRF CLOUD TEST START
FILE(GLOB app_sources src/main.c)
target_sources(app PRIVATE ${app_sources})

# nrf_cloud.c is only included with the poll thread,
# so it needs to be included when the thread is disabled
if (NOT CONFIG_NRF_CLOUD_CONNECTION_POLL_THREAD)
	target_sources(app
		PRIVATE
		${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src/nrf_cloud.c
	)
endif()

target_include_directories(app
	PRIVATE
	src
	${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/include
	${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src
	${ZEPHYR_BASE}/subsys/testsuite/include
	${ZEPHYR_BASE}/../modules/lib/cjson
)

if (CONFIG_NRF_CLOUD_MQTT OR CONFIG_NRF_CLOUD_FOTA OR CONFIG_NRF_MODEM_LIB)

	if (CONFIG_NRF_CLOUD_MQTT OR CONFIG_NRF_CLOUD_FOTA)

		# These code files are mocked and/or not necessary to be included,
		# also prevent nrf_cloud.c to be included multiple times by KConfig
		set_source_files_properties(
			${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src/nrf_cloud.c
			${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src/nrf_cloud_mem.c
			${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src/nrf_cloud_fota.c
			${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src/nrf_cloud_fota_common.c
			${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src/nrf_cloud_codec_internal.c
			${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src/nrf_cloud_fsm.c
			${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/src/nrf_cloud_transport.c
			DIRECTORY ${NRF_SDK_DIR}/subsys/net/lib/nrf_cloud/
			PROPERTIES HEADER_FILE_ONLY ON
		)

		# This code file has functions/structs/variables defined when
		# NET_SOCKETS_POSIX_NAMES=y in zephyr/net/socket.h, so
		# it needs to be excluded when NET_SOCKETS_POSIX_NAMES=n
		set_source_files_properties(
			${NRF_SDK_DIR}/subsys/net/lib/download_client/src/download_client.c
			DIRECTORY ${NRF_SDK_DIR}/subsys/net/lib/download_client/
			PROPERTIES HEADER_FILE_ONLY ON
		)

	endif()

	if (CONFIG_NRF_MODEM_LIB)

		# Similarly, these files need to be excluded when NET_SOCKETS_POSIX_NAMES=n
		set_source_files_properties(
			${NRF_SDK_DIR}/lib/nrf_modem_lib/sanity.c
			${NRF_SDK_DIR}/lib/nrf_modem_lib/nrf91_sockets.c
			DIRECTORY ${NRF_SDK_DIR}/lib/nrf_modem_lib/
			PROPERTIES HEADER_FILE_ONLY ON
		)

	else()

		# This code file uses NRF_MODEM_LIB, should not be included when it is off
		set_source_files_properties(
			${NRF_SDK_DIR}/subsys/dfu/dfu_target/src/dfu_target_modem_delta.c
			DIRECTORY ${NRF_SDK_DIR}/subsys/dfu/dfu_target/
			PROPERTIES HEADER_FILE_ONLY ON
		)

	endif()

endif()
# NRF CLOUD TEST END

##############################################################################
# @file  CMakeLists.txt
# @brief Build configuration of software documentation.
#
# This build configuration builds and/or installs the documentation of the
# software package. The documentation can be divided into user documentation
# (e.g., software manual) and developer documentation (e.g., developer's guide).
# For developers, both those using a library provided by this package and the
# package maintainers, the documentation of the API is of particular interest.
# Developers who are responsible for maintaining this software may be provided
# with even more detailed documentation of the implementation in the so-called
# developer's guide.
#
# See the basis_add_doc() command for details on the documentation build tools
# supported by BASIS for the generation of documentation from plain text files
# and in-source code comments.
#
# Copyright (c) 2012 University of Pennsylvania. All rights reserved.<br />
# See http://www.rad.upenn.edu/sbia/software/license.html or COPYING file.
#
# Contact: SBIA Group <sbia-software at uphs.upenn.edu>
##############################################################################

# Exclude PDF of software manual from distribution package to reduce its size.
list (APPEND CPACK_SOURCE_IGNORE_FILES "doc/DRAMMS_Software_Manual.pdf"
                                       "doc/DRAMMS_Flyer.pdf")

# ============================================================================
# settings
# ============================================================================

# output names of PDF's generated from reStructuredText sources (excl. extension)
set (SITE_OUTPUT_NAME   "${PROJECT_NAME}_Web_Site")
set (MANUAL_OUTPUT_NAME "${PROJECT_NAME}_Software_Manual")
set (GUIDE_OUTPUT_NAME  "${PROJECT_NAME}_Developer_Guide")

# in particular, EXCLUDE_FROM_DOC will be added if pre-generated files exist
set (APIDOC_OPTIONS)
set (SITE_OPTIONS)
set (MANUAL_OPTIONS)
set (GUIDE_OPTIONS)

# ============================================================================
# pre-generated output files
# ============================================================================

# API documentation
if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/apidoc/html/index.html")
  basis_add_doc ("${CMAKE_CURRENT_SOURCE_DIR}/apidoc/html" DESTINATION "${INSTALL_DOC_DIR}/apidoc")
  list (APPEND APIDOC_OPTIONS EXCLUDE_FROM_DOC)
endif ()

# web site
foreach (N IN ITEMS ${SITE_OUTPUT_NAME} site Site)
  if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${N}.pdf")
    basis_add_doc (${N}.pdf OUTPUT_NAME "${SITE_OUTPUT_NAME}.pdf")
    list (APPEND SITE_OPTIONS EXCLUDE_FROM_DOC)
    break ()
  endif ()
endforeach ()

# software manual
foreach (N IN ITEMS ${MANUAL_OUTPUT_NAME} manual Manual UserManual SoftwareManual UserGuide userguide)
  if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${N}.pdf")
    basis_add_doc (${N}.pdf OUTPUT_NAME "${MANUAL_OUTPUT_NAME}.pdf")
    list (APPEND MANUAL_OPTIONS EXCLUDE_FROM_DOC)
    break ()
  endif ()
endforeach ()

# developer's guide
foreach (N IN ITEMS ${GUIDE_OUTPUT_NAME} guide DeveloperGuide DeveloperManual developerguide)
  if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${N}.pdf")
    basis_add_doc (${N}.pdf OUTPUT_NAME "${GUIDE_OUTPUT_NAME}.pdf")
    list (APPEND GUIDE_OPTIONS EXCLUDE_FROM_DOC)
    break ()
  endif ()
endforeach ()

# ============================================================================
# API documentation (in-source code comments)
# ============================================================================

basis_add_doc (
  apidoc           ${APIDOC_OPTIONS}
  GENERATOR        Doxygen
  HTML_DESTINATION "${INSTALL_DOC_DIR}/apidoc"
)

# ============================================================================
# comprehensive manual
# ============================================================================

if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/index.rst")

  basis_add_doc (
    softwaremanual   ${MANUAL_OPTIONS}
    GENERATOR        Sphinx
    BUILDER          pdf html dirhtml man
    MAN_SECTION      7
    HTML_THEME       ${PROJECT_PACKAGE_VENDOR_L}
    RELLINKS         about download installation manual tutorials faq publications people
    SIDEBARWIDTH     280
    HTML_SIDEBARS    searchbox globaltoc
    HTML_STYLE       dramms.css
    HTML_LOGO        logo.png
    HTML_DESTINATION "${INSTALL_DOC_DIR}/html"
    LATEX_TITLE      "${PROJECT_NAME} Software Manual"
    OUTPUT_NAME      "${MANUAL_OUTPUT_NAME}"
    AUTHORS          "Yangming Ou" "Andreas Schuh"
  )

  basis_add_custom_target (site)
  basis_add_custom_target (site_dirhtml)
  basis_add_custom_target (manual)

  basis_add_custom_target (softwaremanual_pdf_update
    COMMAND "${CMAKE_COMMAND}" -E copy_if_different
        "${CMAKE_CURRENT_BINARY_DIR}/latex/DRAMMS_Software_Manual.pdf" 
        "${CMAKE_CURRENT_SOURCE_DIR}/DRAMMS_Software_Manual.pdf"
  )

  basis_add_dependencies (softwaremanual_pdf_update  softwaremanual_pdf)
  basis_add_dependencies (softwaremanual_html        softwaremanual_pdf_update)
  basis_add_dependencies (softwaremanual_dirhtml     softwaremanual_pdf_update)

  basis_add_dependencies  (site         softwaremanual_html)
  basis_add_dependencies  (site_dirhtml softwaremanual_dirhtml)
  basis_add_dependencies  (manual       softwaremanual_pdf_update)

# ============================================================================
# separate web site and manual
# ============================================================================

else ()

  # web site
  if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/site/index.rst")
    basis_add_doc (
      site          ${SITE_OPTIONS}
      GENERATOR     Sphinx
      BUILDER       html dirhtml pdf man
      MAN_SECTION   7
      HTML_THEME    ${PROJECT_PACKAGE_VENDOR_L}
      RELLINKS      download installation manual publications people
      HTML_SIDEBARS searchbox globaltoc
      LATEX_TITLE   "${PROJECT_NAME} Web Site"
      OUTPUT_NAME   "${SITE_OUTPUT_NAME}"
    )
  endif ()

  # software manual
  if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/manual/index.rst")
    basis_add_doc (
      manual           ${MANUAL_OPTIONS}
      GENERATOR        Sphinx
      BUILDER          pdf html
      HTML_DESTINATION "${INSTALL_DOC_DIR}/html"
      LATEX_TITLE      "${PROJECT_NAME} Software Manual"
      OUTPUT_NAME      "${MANUAL_OUTPUT_NAME}"
    )
  endif ()

endif ()

# ============================================================================
# developer's guide
# ============================================================================

if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/guide/index.rst")
  basis_add_doc (
    guide            ${GUIDE_OPTIONS}
    GENERATOR        Sphinx
    BUILDER          pdf html
    HTML_DESTINATION "${INSTALL_DOC_DIR}/guide"
    LATEX_TITLE      "${PROJECT_NAME} Developer's Guide"
    OUTPUT_NAME      "${GUIDE_OUTPUT_NAME}"
  )
endif ()

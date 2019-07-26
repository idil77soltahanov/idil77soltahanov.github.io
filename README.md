# idil77soltahanov.github.io
 line diff
 1.1  --- a/CMakeLists.txt Mon Nov 08 15:14:51 2010 +0000
 1.2  +++ b/CMakeLists.txt Mon Nov 08 21:00:19 2010 +0100
 1.3 @@ -262,6 +262,14 @@
 1.4  MESSAGE(STATUS "Program msgfmt found (${MSGFMT_EXECUTABLE})")
 1.5  ENDIF(MSGFMT_FOUND)
 1.6   
1.7  +IF(WIN32)
 1.8  + # HTML Helpworkshop for creating help file
 1.9 + FIND_PACKAGE(HTMLHelp)
 1.10  + IF(${HTML_HELP_COMPILER} MATCHES "-NOTFOUND")
 1.11  + MESSAGE(FATAL_ERROR "MS HTML Help Workshop not found. It is required to build Hugins help file.")
 1.12 + ENDIF()
 1.13  +ENDIF(WIN32)
 1.14 +
 1.15 ## 1.16  ## LAPACK (optional, enable by -DENABLE_LAPACK=ON)
 1.17 ##
 2.1  --- a/src/hugin1/hugin/MainFrame.cpp Mon Nov 08 15:14:51 2010 +0000
 2.2  +++ b/src/hugin1/hugin/MainFrame.cpp Mon Nov 08 21:00:19 2010 +0100
 2.3 @@ -1206,6 +1206,9 @@
 2.4  
 2.5 DEBUG_TRACE("");
 2.6  
 2.7 +#ifdef __WXMSW__
 2.8  + GetHelpController().DisplaySection(section);
 2.9 +#else
 2.10  #if defined __WXMAC__ && defined MAC_SELF_CONTAINED_BUNDLE
 2.11 // On Mac, xrc/data/help_LOCALE should be in the bundle as LOCALE.lproj/help
 2.12  // which we can rely on the operating sytem to pick the right locale's.
 2.13 @@ -1240,6 +1243,7 @@
 2.14 {
 2.15  wxLogError(_("Can't start system's web browser"));
 2.16 }
 2.17  +#endif
 2.18 }
 2.19   
2.20  void MainFrame::OnTipOfDay(wxCommandEvent& WXUNUSED(e))
 3.1  --- a/src/hugin1/hugin/MainFrame.h Mon Nov 08 15:14:51 2010 +0000
 3.2  +++ b/src/hugin1/hugin/MainFrame.h Mon Nov 08 21:00:19 2010 +0100
 3.3 @@ -32,6 +32,9 @@
 3.4  #include "PT/Panorama.h"
 3.5   
3.6  #include "wx/docview.h"
 3.7  +#ifdef __WXMSW__
 3.8  +#include "wx/msw/helpchm.h"
 3.9  +#endif
 3.10   
3.11  #include "hugin/OptimizePanel.h"
 3.12  #include "hugin/PreferencesDialog.h"
 3.13 @@ -154,6 +157,9 @@
 3.14  struct celeste::svm_model* GetSVMModel();
 3.15      
 3.16 GLPreviewFrame * getGLPreview();
 3.17 +#ifdef __WXMSW__ 3.18  + wxCHMHelpController& GetHelpController() { return m_msHtmlHelp; }
 3.19  +#endif
 3.20   
3.21  protected:
 3.22  // called when a progress message should be displayed
 3.23 @@ -244,6 +250,10 @@
 3.24  double m_progressMax;
 3.25  double m_progress;
 3.26 wxString m_progressMsg;
 3.27 +#ifdef __WXMSW__
 3.28 + wxCHMHelpController m_msHtmlHelp;
 3.29  +#endif
 3.30 +
 3.31  
 3.32 DECLARE_EVENT_TABLE()
 3.33 };
 4.1  --- a/src/hugin1/hugin/huginApp.cpp Mon Nov 08 15:14:51 2010 +0000
 4.2  +++ b/src/hugin1/hugin/huginApp.cpp Mon Nov 08 21:00:19 2010 +0100
 4.3 @@ -56,9 +56,9 @@
 4.4  #include "base_wx/huginConfig.h"
 4.5 #ifdef __WXMSW__
 4.6  #include "wx/dir.h"
 4.7  +#include "wx/cshelp.h"
 4.8 #endif
 4.9  
 4.10 -
 4.11  #include <tiffio.h>
 4.12   
4.13  #include "AboutDialog.h"
 4.14 @@ -137,6 +137,10 @@
 4.15   
4.16   
4.17  #if defined __WXMSW__
 4.18  + // initialize help provider
 4.19 + wxHelpControllerHelpProvider* provider = new wxHelpControllerHelpProvider;
 4.20  + wxHelpProvider::Set(provider);
 4.21 +
 4.22  wxString huginExeDir = getExePath(argv[0]);
 4.23   
4.24  wxString huginRoot;
 4.25 @@ -277,6 +281,8 @@
 4.26  // setup main frame size, after it has been created.
 4.27  RestoreFramePosition(frame, wxT("MainFrame"));
 4.28 #ifdef __WXMSW__
 4.29  + provider->SetHelpController(&frame->GetHelpController());
 4.30  + frame->GetHelpController().Initialize(m_xrcPrefix+wxT("data/hugin_help_en_EN.chm"));
 4.31 frame->SendSizeEvent();
 4.32 #endif
 4.33  
 5.1  --- a/src/hugin1/hugin/xrc/data/help_en_EN/CMakeLists.txt Mon Nov 08 15:14:51 2010 +0000
 5.2  +++ b/src/hugin1/hugin/xrc/data/help_en_EN/CMakeLists.txt Mon Nov 08 21:00:19 2010 +0100
 5.3 @@ -1,6 +1,47 @@
 5.4 +
 5.5  +# install help/manual
 5.6  +IF(WIN32)
 5.7 +
 5.8  +# running hhc with relative path does not work correctly, so we copy all manual file into temporary
 5.9  +# directory and run hhc there
 5.10 +
 5.11  +FILE(GLOB HELPFILES 
 5.12  + RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
 5.13 + *.htm *.html *.ico *.gif *.JPG *.jpg *.png *.xpm hugin_help_en_EN.hhp help_index_en.hhk help_toc_en.hhc hhc.cmake
 5.14 +)
 5.15 +
 5.16  +SET(HELP_DIR ${CMAKE_BINARY_DIR}/help)
 5.17  +SET(HELPFILES2) # empty list
 5.18 +
 5.19  +FOREACH(_file ${HELPFILES})
 5.20 + ADD_CUSTOM_COMMAND(
 5.21  + OUTPUT "${HELP_DIR}/${_file}"
 5.22  + COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_SOURCE_DIR}/${_file}" "${HELP_DIR}/${_file}"
 5.23  + DEPENDS ${_file}
 5.24  + COMMENT "Copy ${_file} to ${HELP_DIR}"
 5.25 + )
 5.26 + SET_SOURCE_FILES_PROPERTIES("${HELP_DIR}/${_file}" GENERATED)
 5.27  + LIST(APPEND HELPFILES2 "${HELP_DIR}/${_file}")
 5.28 +ENDFOREACH()
 5.29 +
 5.30 +ADD_CUSTOM_COMMAND(
 5.31  + OUTPUT ${HELP_DIR}/hugin_help_en_EN.chm
 5.32  + COMMAND ${CMAKE_COMMAND} -DHTML_HELP_COMPILER=${HTML_HELP_COMPILER} -P hhc.cmake
 5.33  + # COMMAND ${HTML_HELP_COMPILER} hugin_help_en_EN.hhp
 5.34  + DEPENDS ${HELPFILES2}
 5.35 + WORKING_DIRECTORY ${HELP_DIR}
 5.36  + COMMENT "Building help file"
 5.37 +)
 5.38  +SET_SOURCE_FILES_PROPERTIES("${HELP_DIR}/hugin_help_en_EN.chm" GENERATED) 5.39  +ADD_CUSTOM_TARGET(help ALL DEPENDS ${HELPFILE2} ${HELP_DIR}/hugin_help_en_EN.chm )
 5.40 +
 5.41  +INSTALL(FILES ${CMAKE_BINARY_DIR}/help/hugin_help_en_EN.chm DESTINATION ${HUGINDATADIR}/xrc/data/)
 5.42 +
 5.43  +ELSE(WIN32)
 5.44   
5.45  FILE(GLOB DATAFILES *.htm *.html *.ico *.gif *.JPG *.jpg *.png *.xpm *.hhc *.hhk *.hhp
 5.46 *.manual)
 5.47   
5.48  INSTALL(FILES ${DATAFILES} DESTINATION ${HUGINDATADIR}/xrc/data/help_en_EN)
 5.49   
5.50  +ENDIF(WIN32)
 6.1  --- /dev/null Thu Jan 01 00:00:00 1970 +0000
 6.2  +++ b/src/hugin1/hugin/xrc/data/help_en_EN/help_index_en.hhk Mon Nov 08 21:00:19 2010 +0100
 6.3 @@ -0,0 +1,789 @@
 6.4 +<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML//EN">
 6.5  +<HTML>
 6.6  +<HEAD>
 6.7  +<meta name="GENERATOR" content="Microsoft&reg; HTML Help Workshop 4.1">
 6.8 +<!-- Sitemap 1.0 -->
 6.9  +</HEAD><BODY>
 6.10 +<UL>
 6.11  + <LI> <OBJECT type="text/sitemap">
 6.12  + <param name="Name" value="16 bit">
 6.13  + <param name="Name" value="16 bit">
 6.14 + html">
 6.15  + <param name="Name" value="16 bit workflow with hugin">
 6.16 + <param name="Local" value="16bit_workflow_with_hugin.html">
 6.17  + </OBJECT>
 6.18  + <LI> <OBJECT type="text/sitemap">
 6.19  + <param name="Name" value="Aliasing">
 6.20  + <param name="Name" value="Aliasing">
 6.21 + <param name="Local" value="Aliasing.html">
 6.22  + </OBJECT>
 6.23  + <LI> <OBJECT type="text/sitemap">
 6.24  + <param name="Name" value="Align a stack of photo">
 6.25  + <param name="Name" value="Align a stack of photos">
 6.26 + html">
 6.27  + <param name="Name" value="Align image stack">
 6.28 + <param name="Local" value="Align_image_stack.html">
 6.29  + </OBJECT>
 6.30  + <LI> <OBJECT type="text/sitemap">
 6.31  + <param name="Name" value="Align_image_stack">
 6.32  + <param name="Name" value="Align image stack">
 6.33 + <param name="Local" value="Align_image_stack.html">
 6.34  + <param name="Name" value="Align a stack of photos">
 6.35 + <param name="Local" value="Align_a_stack_of_photos.html">
 6.36  + </OBJECT>
 6.37 + <LI> <OBJECT type="text/sitemap">
 6.38  + <param name="Name" value="Alpha channel">
 6.39  + <param name="Name" value="Alpha channel">
 6.40 + <param name="Local" value="Alpha_channel.html">
 6.41  + </OBJECT>
 6.42  + <LI> <OBJECT type="text/sitemap">
 6.43  + <param name="Name" value="Aspect ratio">
 6.44  + <param name="Name" value="Aspect Ratio">
 6.45 + <param name="Local" value="Aspect_Ratio.html">
 6.46 + <param name="URL" value="Portrait.html">
 6.47  + </OBJECT>
 6.48  + <LI> <OBJECT type="text/sitemap">
 6.49  + <param name="Name" value="Assistant"> 6.50  + <param name="Name" value="Hugin Assistant tab">
 6.51 + <param name="Local" value="Hugin_Assistant_tab.html">
 6.52  + </OBJECT>
 6.53  + <LI> <OBJECT type="text/sitemap">
 6.54  + <param name="Name" value="Autooptimiser">
 6.55  + <param name="Name" value="Autooptimiser">
 6.56 + <param name="Local" value="Autooptimiser.html">
 6.57  + </OBJECT>
 6.58  + <LI> <OBJECT type="text/sitemap">
 6.59  + <param name="Name" value="Autopano">
 6.60  + <param name="Name" value="Autopano">
 6.61 + <param name="Local" value="Autopano.html">
 6.62 + <param name="Name" value="Autopano-sift">
 6.63 + <param name="Local" value="Autopano-sift.html">
 6.64  + <param name="Name" value="Autopano-sift-C">
 6.65 + <param name="Local" value="Autopano-sift-C.html">
 6.66  + </OBJECT>
 6.67  + <LI> <OBJECT type="text/sitemap">
 6.68  + <param name="Name" value="Autopano-SIFT">
 6.69  + <param name="Name" value="Autopano-sift">
 6.70 + <param name="Local" value="Autopano-sift.html">
 6.71  + <param name="Name" value="Autopano-sift-C">
 6.72 + <param name="Local" value="Autopano-sift-C.html">
 6.73  + </OBJECT>
 6.74 + <LI> <OBJECT type="text/sitemap">
 6.75  + <param name="Name" value="Autopano-SIFT-C">
 6.76  + <param name="Name" value="Autopano-sift-C">
 6.77 + <param name="Local" value="Autopano-sift-C.html">
 6.78  + </OBJECT>
 6.79  + <LI> <OBJECT type="text/sitemap">
 6.80  + <param name="Name" value="Banding">
 6.81  + <param name="Name" value="Banding">
 6.82 + <param name="Local" value="Banding.html">
 6.83  + </OBJECT>
 6.84  + <LI> <OBJECT type="text/sitemap">
 6.85  + <param name="Name" value="Barrel distortion">
 6.86  + <param name="Name" value="Barrel distortion"> 6.87 + <param name="Local" value="Barrel_distortion.html">
 6.88  + </OBJECT>
 6.89  + <LI> <OBJECT type="text/sitemap">
 6.90  + <param name="Name" value="Batch processor">
 6.91  + <param name="Name" value="Hugin Batch Processor">
 6.92 + <param name="Local" value="Hugin_Batch_Processor.html">
 6.93  + </OBJECT>
 6.94  + <LI> <OBJECT type="text/sitemap">
 6.95  + <param name="Name" value="Bracketing">
 6.96  + <param name="Name" value="Bracketing">
 6.97 + <param name="Local" value="Bracketing.html">
 6.98  + </OBJECT>
 6.99  + <LI> <OBJECT type="text/sitemap"> 6.100  + <param name="Name" value="Bugs">
 6.101  + <param name="Name" value="Hugin Trackers">
 6.102 + <param name="Local" value="Hugin_Trackers.html">
 6.103  + </OBJECT>
 6.104  + <LI> <OBJECT type="text/sitemap">
 6.105  + <param name="Name" value="Camera and lens tab">
 6.106  + <param name="Name" value="Hugin Camera and Lens tab">
 6.107 + <param name="Local" value="Hugin_Camera_and_Lens_tab.html">
 6.108  + </OBJECT>
 6.109  + <LI> <OBJECT type="text/sitemap">
 6.110  + <param name="Name" value="Camera response curve">
 6.111 + <param name="Name" value="Camera response curve">
 6.112 + <param name="Local" value="Camera_response_curve.html">
 6.113  + </OBJECT>
 6.114  + <LI> <OBJECT type="text/sitemap">
 6.115  + <param name="Name" value="CCD">
 6.116  + <param name="Name" value="CCD">
 6.117 + <param name="Local" value="CCD.html">
 6.118  + </OBJECT>
 6.119  + <LI> <OBJECT type="text/sitemap">
 6.120  + <param name="Name" value="Celeste">
 6.121  + <param name="Name" value="Celeste standalone">
 6.122 + <param name="Local" value="Celeste_standalone.html">
 6.123 + <param name="Name" value="Hugin Images tab">
 6.124 + <param name="Local" value="Hugin_Images_tab.html">
 6.125  + <param name="Name" value="Using Celeste with hugin">
 6.126 + <param name="Local" value="Using_Celeste_with_hugin.html">
 6.127  + </OBJECT>
 6.128  + <LI> <OBJECT type="text/sitemap">
 6.129  + <param name="Name" value="Celeste_standalone">
 6.130  + <param name="Name" value="Celeste standalone">
 6.131 + <param name="Local" value="Celeste_standalone.html">
 6.132  + </OBJECT>
 6.133  + <LI> <OBJECT type="text/sitemap">
 6.134 + <param name="Name" value="Chromatic aberration">
 6.135  + <param name="Name" value="Chromatic aberration">
 6.136 + <param name="Local" value="Chromatic_aberration.html">
 6.137  + </OBJECT>
 6.138  + <LI> <OBJECT type="text/sitemap">
 6.139  + <param name="Name" value="Color correct tiff">
 6.140  + <param name="Name" value="Color correct tiff">
 6.141 + <param name="Local" value="Color_correct_tiff.html">
 6.142  + </OBJECT>
 6.143  + <LI> <OBJECT type="text/sitemap">
 6.144  + <param name="Name" value="Colour profile">
 6.145  + <param name="Name" value="Colour profile"> 6.146 + <param name="Local" value="Colour_profile.html">
 6.147  + </OBJECT>
 6.148  + <LI> <OBJECT type="text/sitemap">
 6.149  + <param name="Name" value="Contrast">
 6.150  + <param name="Name" value="Contrast">
 6.151 + <param name="Local" value="Contrast.html">
 6.152  + </OBJECT>
 6.153  + <LI> <OBJECT type="text/sitemap">
 6.154  + <param name="Name" value="Control point">
 6.155  + <param name="Name" value="Control points">
 6.156 + <param name="Local" value="Control_points.html">
 6.157  + <param name="Name" value="Horizontal control points">
 6.158 + html">
 6.159  + <param name="Name" value="Vertical control points">
 6.160 + <param name="Local" value="Vertical_control_points.html">
 6.161  + <param name="Name" value="Straight line control points">
 6.162 + <param name="Local" value="Straight_line_control_points.html">
 6.163  + </OBJECT>
 6.164  + <LI> <OBJECT type="text/sitemap">
 6.165  + <param name="Name" value="Control point detector">
 6.166  + <param name="Name" value="Control point generators">
 6.167 + <param name="Local" value="Control_point_generators.html"> 6.168  + <param name="Name" value="Autopano">
 6.169 + <param name="Local" value="Autopano.html">
 6.170  + <param name="Name" value="Autopano-sift">
 6.171 + <param name="Local" value="Autopano-sift.html">
 6.172  + <param name="Name" value="Autopano-sift-C">
 6.173 + <param name="Local" value="Autopano-sift-C.html">
 6.174  + <param name="Name" value="Panomatic">
 6.175 + <param name="Local" value="Panomatic.html">
 6.176  + <param name="Name" value="Cpfind">
 6.177 + <param name="Local" value="Cpfind.html">
 6.178  + <param name="Name" value="Align image stack">
 6.179 + html">
 6.180  + <param name="Name" value="MatchPoint">
 6.181 + <param name="Local" value="MatchPoint.html">
 6.182  + </OBJECT>
 6.183  + <LI> <OBJECT type="text/sitemap">
 6.184  + <param name="Name" value="Control point detector parameter">
 6.185  + <param name="Name" value="Control Point Detector Parameters">
 6.186 + <param name="Local" value="Control_Point_Detector_Parameters.html">
 6.187  + <param name="Name" value="Hugin Parameters for Control Point Detectors dialog">
 6.188 + html">
 6.189  + </OBJECT>
 6.190  + <LI> <OBJECT type="text/sitemap">
 6.191  + <param name="Name" value="Control point generator">
 6.192  + <param name="Name" value="Control point generators">
 6.193 + <param name="Local" value="Control_point_generators.html">
 6.194  + <param name="Name" value="Autopano">
 6.195 + <param name="Local" value="Autopano.html">
 6.196  + <param name="Name" value="Autopano-sift">
 6.197 + <param name="Local" value="Autopano-sift.html">
 6.198 + <param name="Name" value="Autopano-sift-C">
 6.199 + <param name="Local" value="Autopano-sift-C.html">
 6.200  + <param name="Name" value="Panomatic">
 6.201 + <param name="Local" value="Panomatic.html">
 6.202  + <param name="Name" value="Align image stack">
 6.203 + <param name="Local" value="Align_image_stack.html">
 6.204  + <param name="Name" value="Cpfind">
 6.205 + <param name="Local" value="Cpfind.html">
 6.206  + </OBJECT>
 6.207  + <LI> <OBJECT type="text/sitemap">
 6.208  + <param name="Name" value="Control points tab">
 6.209 + <param name="Name" value="Hugin Control Points tab">
 6.210 + <param name="Local" value="Hugin_Control_Points_tab.html">
 6.211  + </OBJECT>
 6.212  + <LI> <OBJECT type="text/sitemap">
 6.213  + <param name="Name" value="Control points table">
 6.214  + <param name="Name" value="Hugin Control Points table">
 6.215 + <param name="Local" value="Hugin_Control_Points_table.html">
 6.216  + </OBJECT>
 6.217  + <LI> <OBJECT type="text/sitemap">
 6.218  + <param name="Name" value="Cpclean">
 6.219  + <param name="Name" value="Cpclean">
 6.220 + html">
 6.221  + </OBJECT>
 6.222  + <LI> <OBJECT type="text/sitemap">
 6.223  + <param name="Name" value="Cpfind">
 6.224  + <param name="Name" value="Cpfind">
 6.225 + <param name="Local" value="Cpfind.html">
 6.226  + </OBJECT>
 6.227  + <LI> <OBJECT type="text/sitemap">
 6.228  + <param name="Name" value="Crop factor">
 6.229  + <param name="Name" value="Crop factor">
 6.230 + <param name="Local" value="Crop_factor.html">
 6.231  + </OBJECT>
 6.232  + <LI> <OBJECT type="text/sitemap">
 6.233  + <param name="Name" value="Crop tab">
 6.234 + <param name="Name" value="Hugin Crop tab">
 6.235 + <param name="Local" value="Hugin_Crop_tab.html">
 6.236  + </OBJECT>
 6.237  + <LI> <OBJECT type="text/sitemap">
 6.238  + <param name="Name" value="Cropped TIFF">
 6.239  + <param name="Name" value="Cropped TIFF">
 6.240 + <param name="Local" value="Cropped_TIFF.html">
 6.241  + </OBJECT>
 6.242  + <LI> <OBJECT type="text/sitemap">
 6.243  + <param name="Name" value="Cubic projection">
 6.244  + <param name="Name" value="Cubic Projection">
 6.245 + <param name="Local" value="Cubic_Projection.html">
 6.246  + </OBJECT> 6.247  + <LI> <OBJECT type="text/sitemap">
 6.248  + <param name="Name" value="Cylindrical projection">
 6.249  + <param name="Name" value="Cylindrical Projection">
 6.250 + <param name="Local" value="Cylindrical_Projection.html">
 6.251  + </OBJECT>
 6.252  + <LI> <OBJECT type="text/sitemap">
 6.253  + <param name="Name" value="Depth of field">
 6.254  + <param name="Name" value="Depth of Field">
 6.255 + <param name="Local" value="Depth_of_Field.html">
 6.256  + </OBJECT>
 6.257  + <LI> <OBJECT type="text/sitemap">
 6.258  + <param name="Name" value="Distortion">
 6.259 + <param name="Name" value="Lens distortion">
 6.260 + <param name="Local" value="Lens_distortion.html">
 6.261  + <param name="Name" value="Barrel distortion">
 6.262 + <param name="Local" value="Barrel_distortion.html">
 6.263  + <param name="Name" value="Pincushion distortion">
 6.264 + <param name="Local" value="Pincushion_distortion.html">
 6.265  + <param name="Name" value="Wavy distortion">
 6.266 + <param name="Local" value="Wavy_distortion.html">
 6.267  + </OBJECT>
 6.268  + <LI> <OBJECT type="text/sitemap">
 6.269  + <param name="Name" value="DPI">
 6.270 + <param name="Name" value="DPI">
 6.271 + <param name="Local" value="PPI.html">
 6.272  + </OBJECT>
 6.273  + <LI> <OBJECT type="text/sitemap">
 6.274  + <param name="Name" value="DSLR sperical resolution">
 6.275  + <param name="Name" value="DSLR spherical resolution">
 6.276 + <param name="Local" value="DSLR_spherical_resolution.html">
 6.277  + </OBJECT>
 6.278  + <LI> <OBJECT type="text/sitemap">
 6.279  + <param name="Name" value="Dust removal with flatfield">
 6.280  + <param name="Name" value="Dust Removal with Flatfield">
 6.281 + html">
 6.282  + </OBJECT>
 6.283  + <LI> <OBJECT type="text/sitemap">
 6.284  + <param name="Name" value="Dynamic range">
 6.285  + <param name="Name" value="Dynamic range">
 6.286 + <param name="Local" value="Dynamic_range.html">
 6.287  + <param name="Name" value="HDR">
 6.288 + <param name="Local" value="HDR.html">
 6.289  + </OBJECT>
 6.290  + <LI> <OBJECT type="text/sitemap">
 6.291  + <param name="Name" value="Enblend">
   6.292+ <param name="Name" value="Enblend">
 6.293 + <param name="Local" value="Enblend.html"> 6.294  + <param name="Name" value="Enblend reference manual">
 6.295 + <param name="Local" value="Enblend_reference_manual.html">
 6.296  + </OBJECT>
 6.297  + <LI> <OBJECT type="text/sitemap">
 6.298  + <param name="Name" value="Enfuse">
 6.299  + <param name="Name" value="Enfuse">
 6.300 + <param name="Local" value="Enfuse.html">
 6.301  + <param name="Name" value="Enfuse reference manual">
 6.302 + <param name="Local" value="Enfuse_reference_manual.html">
 6.303  + </OBJECT>
 6.304  + <LI> <OBJECT type="text/sitemap">
 6.305 + <param name="Name" value="Equirectangular projection">
 6.306  + <param name="Name" value="Equirectangular Projection">
 6.307 + <param name="Local" value="Equirectangular_Projection.html">
 6.308  + </OBJECT>
 6.309  + <LI> <OBJECT type="text/sitemap">
 6.310  + <param name="Name" value="EXIF">
 6.311  + <param name="Name" value="EXIF">
 6.312 + <param name="Local" value="EXIF.html">
 6.313  + </OBJECT>
 6.314  + <LI> <OBJECT type="text/sitemap">
 6.315  + <param name="Name" value="Exposure tab">
 6.316  + <param name="Name" value="Hugin Exposure tab">
 6.317 + html">
 6.318  + </OBJECT>
 6.319  + <LI> <OBJECT type="text/sitemap">
 6.320  + <param name="Name" value="FAQ">
 6.321  + <param name="Name" value="Hugin FAQ">
 6.322 + <param name="Local" value="Hugin_FAQ.html">
 6.323  + </OBJECT>
 6.324  + <LI> <OBJECT type="text/sitemap">
 6.325  + <param name="Name" value="Fast preview window">
 6.326  + <param name="Name" value="Hugin Fast Preview window">
 6.327 + <param name="Local" value="Hugin_Fast_Preview_window.html">
 6.328  + </OBJECT>
 6.329  + <LI> <OBJECT type="text/sitemap"> 6.330  + <param name="Name" value="Field of view">
 6.331  + <param name="Name" value="Field of View">
 6.332 + <param name="Local" value="Field_of_View.html">
 6.333  + </OBJECT>
 6.334  + <LI> <OBJECT type="text/sitemap">
 6.335  + <param name="Name" value="Fisheye projection">
 6.336  + <param name="Name" value="Fisheye Projection">
 6.337 + <param name="Local" value="Fisheye_Projection.html">
 6.338  + </OBJECT>
 6.339  + <LI> <OBJECT type="text/sitemap">
 6.340  + <param name="Name" value="Focal length">
 6.341  + <param name="Name" value="Focal Length">
 6.342 + html">
 6.343  + </OBJECT>
 6.344  + <LI> <OBJECT type="text/sitemap">
 6.345  + <param name="Name" value="Freepv">
 6.346  + <param name="Name" value="Freepv">
 6.347 + <param name="Local" value="Freepv.html">
 6.348  + </OBJECT>
 6.349  + <LI> <OBJECT type="text/sitemap">
 6.350  + <param name="Name" value="Fulla">
 6.351  + <param name="Name" value="Fulla">
 6.352 + <param name="Local" value="Fulla.html">
 6.353  + </OBJECT>
 6.354  + <LI> <OBJECT type="text/sitemap">
 6.355  + <param name="Name" value="Gamma">
 6.356 + <param name="Name" value="Gamma">
 6.357 + <param name="Local" value="Gamma.html">
 6.358  + </OBJECT>
 6.359  + <LI> <OBJECT type="text/sitemap">
 6.360  + <param name="Name" value="General Panini projection">
 6.361  + <param name="Name" value="The General Panini Projection">
 6.362 + <param name="Local" value="The_General_Panini_Projection.html">
 6.363  + </OBJECT>
 6.364  + <LI> <OBJECT type="text/sitemap">
 6.365  + <param name="Name" value="GIF">
 6.366  + <param name="Name" value="GIF">
 6.367 + <param name="Local" value="GIF.html">
 6.368  + </OBJECT>
 6.369 + <LI> <OBJECT type="text/sitemap">
 6.370  + <param name="Name" value="HDR">
 6.371  + <param name="Name" value="HDR">
 6.372 + <param name="Local" value="HDR.html">
 6.373  + <param name="Name" value="HDR workflow with hugin">
 6.374 + <param name="Local" value="HDR_workflow_with_hugin.html">
 6.375  + </OBJECT>
 6.376  + <LI> <OBJECT type="text/sitemap">
 6.377  + <param name="Name" value="Horizontal control point">
 6.378  + <param name="Name" value="Horizontal control points">
 6.379 + <param name="Local" value="Horizontal_control_points.html">
 6.380  + </OBJECT>
 6.381 + <LI> <OBJECT type="text/sitemap">
 6.382  + <param name="Name" value="Hugin">
 6.383  + <param name="Name" value="Hugin">
 6.384 + <param name="Local" value="Hugin.html">
 6.385  + </OBJECT>
 6.386  + <LI> <OBJECT type="text/sitemap">
 6.387  + <param name="Name" value="Icpfind">
 6.388  + <param name="Name" value="Icpfind">
 6.389 + <param name="Local" value="Icpfind.html">
 6.390  + </OBJECT>
 6.391  + <LI> <OBJECT type="text/sitemap">
 6.392  + <param name="Name" value="Image format">
 6.393  + <param name="Name" value="GIF">
 6.394 + <param name="Local" value="GIF.html"> 6.395  + <param name="Name" value="JPEG">
 6.396 + <param name="Local" value="JPEG.html">
 6.397  + <param name="Name" value="TIFF">
 6.398 + <param name="Local" value="TIFF.html">
 6.399  + <param name="Name" value="PNG">
 6.400 + <param name="Local" value="PNG.html">
 6.401  + <param name="Name" value="OpenEXR">
 6.402 + <param name="Local" value="OpenEXR.html">
 6.403  + <param name="Name" value="PSD">
 6.404 + <param name="Local" value="PSD.html">
 6.405  + </OBJECT>
 6.406  + <LI> <OBJECT type="text/sitemap">
 6.407  + <param name="Name" value="Images tab">
 6.408 + <param name="Name" value="Hugin Images tab">
 6.409 + <param name="Local" value="Hugin_Images_tab.html">
 6.410  + </OBJECT>
 6.411  + <LI> <OBJECT type="text/sitemap">
 6.412  + <param name="Name" value="Interpolation">
 6.413  + <param name="Name" value="Interpolation">
 6.414 + <param name="Local" value="Interpolation.html">
 6.415  + </OBJECT>
 6.416  + <LI> <OBJECT type="text/sitemap">
 6.417  + <param name="Name" value="JPEG">
 6.418  + <param name="Name" value="JPEG">
 6.419 + <param name="Local" value="JPEG.html">
 6.420  + </OBJECT>
 6.421 + <LI> <OBJECT type="text/sitemap">
 6.422  + <param name="Name" value="Keyboard shortcuts">
 6.423  + <param name="Name" value="Hugin Keyboard shortcuts">
 6.424 + <param name="Local" value="Hugin_Keyboard_shortcuts.html">
 6.425  + </OBJECT>
 6.426  + <LI> <OBJECT type="text/sitemap">
 6.427  + <param name="Name" value="Landscape">
 6.428  + <param name="Name" value="Landscape">
 6.429 + <param name="Local" value="Landscape.html">
 6.430  + </OBJECT>
 6.431  + <LI> <OBJECT type="text/sitemap">
 6.432  + <param name="Name" value="Lens correction model">
 6.433 + <param name="Name" value="Lens correction model">
 6.434 + <param name="Local" value="Lens_correction_model.html">
 6.435  + </OBJECT>
 6.436  + <LI> <OBJECT type="text/sitemap">
 6.437  + <param name="Name" value="Lens distortion">
 6.438  + <param name="Name" value="Lens distortion">
 6.439 + <param name="Local" value="Lens_distortion.html">
 6.440  + <param name="Name" value="Barrel distortion">
 6.441 + <param name="Local" value="Barrel_distortion.html">
 6.442  + <param name="Name" value="Pincushion distortion">
 6.443 + html">
 6.444  + <param name="Name" value="Wavy distortion">
 6.445 + <param name="Local" value="Wavy_distortion.html">
 6.446  + </OBJECT>
 6.447  + <LI> <OBJECT type="text/sitemap">
 6.448  + <param name="Name" value="Lightprobe">
 6.449  + <param name="Name" value="Lightprobe">
 6.450 + <param name="Local" value="Lightprobe.html">
 6.451  + </OBJECT>
 6.452  + <LI> <OBJECT type="text/sitemap">
 6.453  + <param name="Name" value="Main window">
 6.454  + <param name="Name" value="Hugin Main window">
 6.455 + html">
 6.456  + </OBJECT>
 6.457  + <LI> <OBJECT type="text/sitemap">
 6.458  + <param name="Name" value="Mask tab">
 6.459  + <param name="Name" value="Hugin Mask tab">
 6.460 + <param name="Local" value="Hugin_Mask_tab.html">
 6.461  + </OBJECT>
 6.462  + <LI> <OBJECT type="text/sitemap">
 6.463  + <param name="Name" value="Matchpoint">
 6.464  + <param name="Name" value="MatchPoint">
 6.465 + <param name="Local" value="MatchPoint.html">
 6.466  + </OBJECT>
 6.467  + <LI> <OBJECT type="text/sitemap">
 6.468 + <param name="Name" value="Nadir">
 6.469  + <param name="Name" value="Nadir">
 6.470 + <param name="Local" value="Nadir.html">
 6.471  + </OBJECT>
 6.472  + <LI> <OBJECT type="text/sitemap">
 6.473  + <param name="Name" value="Nona">
 6.474  + <param name="Name" value="Nona">
 6.475 + <param name="Local" value="Nona.html">
 6.476  + </OBJECT>
 6.477  + <LI> <OBJECT type="text/sitemap">
   6.478+ <param name="Name" value="Nona GUI">
 6.479  + <param name="Name" value="Nona gui">
 6.480 + <param name="Local" value="Nona_gui.html">
 6.481  + </OBJECT>
 6.482 + <LI> <OBJECT type="text/sitemap">
 6.483  + <param name="Name" value="No-parallax point">
 6.484  + <param name="Name" value="No-parallax point">
 6.485 + <param name="Local" value="No-parallax_point.html">
 6.486  + </OBJECT>
 6.487  + <LI> <OBJECT type="text/sitemap">
 6.488  + <param name="Name" value="NPP">
 6.489  + <param name="Name" value="No-parallax point">
 6.490 + <param name="Local" value="No-parallax_point.html">
 6.491  + </OBJECT>
 6.492  + <LI> <OBJECT type="text/sitemap">
 6.493  + <param name="Name" value="OpenEXR">
 6.494  + <param name="Name" value="OpenEXR"> 6.495 + <param name="Local" value="OpenEXR.html">
 6.496  + </OBJECT>
 6.497  + <LI> <OBJECT type="text/sitemap">
 6.498  + <param name="Name" value="Optimization">
 6.499  + <param name="Name" value="Optimization">
 6.500 + <param name="Local" value="Optimization.html">
 6.501  + </OBJECT>
 6.502  + <LI> <OBJECT type="text/sitemap">
 6.503  + <param name="Name" value="Optimizer tab">
 6.504  + <param name="Name" value="Hugin Optimizer tab">
 6.505 + <param name="Local" value="Hugin_Optimizer_tab.html">
 6.506  + </OBJECT>
 6.507  + <LI> <OBJECT type="text/sitemap">
 6.508 + <param name="Name" value="Panini projection">
 6.509  + <param name="Name" value="The General Panini Projection">
 6.510 + <param name="Local" value="The_General_Panini_Projection.html">
 6.511  + </OBJECT>
 6.512  + <LI> <OBJECT type="text/sitemap">
 6.513  + <param name="Name" value="Pano_modify">
 6.514  + <param name="Name" value="Pano modify">
 6.515 + <param name="Local" value="Pano_modify.html">
 6.516  + </OBJECT>
 6.517  + <LI> <OBJECT type="text/sitemap">
 6.518  + <param name="Name" value="Pano12">
 6.519  + <param name="Name" value="Pano12">
 6.520 + html">
 6.521  + </OBJECT>
 6.522  + <LI> <OBJECT type="text/sitemap">
 6.523  + <param name="Name" value="Panoglview">
 6.524  + <param name="Name" value="Panoglview">
 6.525 + <param name="Local" value="Panoglview.html">
 6.526  + </OBJECT>
 6.527  + <LI> <OBJECT type="text/sitemap">
 6.528  + <param name="Name" value="Panoinfo">
 6.529  + <param name="Name" value="Panoinfo">
 6.530 + <param name="Local" value="Panoinfo.html">
 6.531  + </OBJECT>
 6.532  + <LI> <OBJECT type="text/sitemap">
 6.533  + <param name="Name" value="Panomatic"> 6.534  + <param name="Name" value="Panomatic">
 6.535 + <param name="Local" value="Panomatic.html">
 6.536  + </OBJECT>
 6.537  + <LI> <OBJECT type="text/sitemap">
 6.538  + <param name="Name" value="Panorama">
 6.539  + <param name="Name" value="Panorama">
 6.540 + <param name="Local" value="Panorama.html">
 6.541  + </OBJECT>
 6.542  + <LI> <OBJECT type="text/sitemap">
 6.543  + <param name="Name" value="Panorama formats">
 6.544  + <param name="Name" value="Panorama formats">
 6.545 + <param name="Local" value="Panorama_formats.html">
 6.546  + </OBJECT>
 6.547 + <LI> <OBJECT type="text/sitemap">
 6.548  + <param name="Name" value="Panorama tools">
 6.549  + <param name="Name" value="Panorama tools">
 6.550 + <param name="Local" value="Panorama_tools.html">
 6.551  + </OBJECT>
 6.552  + <LI> <OBJECT type="text/sitemap">
 6.553  + <param name="Name" value="Panotools">
 6.554  + <param name="Name" value="Panorama tools">
 6.555 + <param name="Local" value="Panorama_tools.html">
 6.556  + </OBJECT>
 6.557  + <LI> <OBJECT type="text/sitemap">
 6.558  + <param name="Name" value="Parallax">
 6.559  + <param name="Name" value="Parallax"> 6.560 + <param name="Local" value="Parallax.html">
 6.561  + </OBJECT>
 6.562  + <LI> <OBJECT type="text/sitemap">
 6.563  + <param name="Name" value="Perspective correction">
 6.564  + <param name="Name" value="Perspective correction">
 6.565 + <param name="Local" value="Perspective_correction.html">
 6.566  + </OBJECT>
 6.567  + <LI> <OBJECT type="text/sitemap">
 6.568  + <param name="Name" value="Perspective distortion">
 6.569  + <param name="Name" value="Perspective distortion">
 6.570 + <param name="Local" value="Perspective_distortion.html">
 6.571  + </OBJECT>
 6.572 + <LI> <OBJECT type="text/sitemap">
 6.573  + <param name="Name" value="Pfstmo">
 6.574  + <param name="Name" value="Pfstmo">
 6.575 + <param name="Local" value="Pfstmo.html">
 6.576  + </OBJECT>
 6.577  + <LI> <OBJECT type="text/sitemap">
 6.578  + <param name="Name" value="Pincushion distortion">
 6.579  + <param name="Name" value="Pincushion distortion">
 6.580 + <param name="Local" value="Pincushion_distortion.html">
 6.581  + </OBJECT>
 6.582  + <LI> <OBJECT type="text/sitemap">
 6.583  + <param name="Name" value="Pitch">
 6.584  + <param name="Name" value="Pitch">
 6.585 + html">
 6.586  + </OBJECT>
 6.587  + <LI> <OBJECT type="text/sitemap">
 6.588  + <param name="Name" value="PNG">
 6.589  + <param name="Name" value="PNG">
 6.590 + <param name="Local" value="PNG.html">
 6.591  + </OBJECT>
 6.592  + <LI> <OBJECT type="text/sitemap">
 6.593  + <param name="Name" value="Portrait">
 6.594  + <param name="Name" value="Portrait">
 6.595 + <param name="Local" value="Portrait.html">
 6.596  + </OBJECT>
 6.597  + <LI> <OBJECT type="text/sitemap">
 6.598  + <param name="Name" value="Preferences">
 6.599 + <param name="Name" value="Hugin Preferences">
 6.600 + <param name="Local" value="Hugin_Preferences.html">
 6.601  + </OBJECT>
 6.602  + <LI> <OBJECT type="text/sitemap">
 6.603  + <param name="Name" value="Preview window">
 6.604  + <param name="Name" value="Hugin Preview window">
 6.605 + <param name="Local" value="Hugin_Preview_window.html">
 6.606  + </OBJECT>
 6.607  + <LI> <OBJECT type="text/sitemap">
 6.608  + <param name="Name" value="Projection">
 6.609  + <param name="Name" value="Projections">
 6.610 + <param name="Local" value="Projections.html">
 6.611 + <param name="Name" value="Cubic Projection">
 6.612 + <param name="Local" value="Cubic_Projection.html">
 6.613  + <param name="Name" value="Cylindrical Projection">
 6.614 + <param name="Local" value="Cylindrical_Projection.html">
 6.615  + <param name="Name" value="Equirectangular Projection">
 6.616 + <param name="Local" value="Equirectangular_Projection.html">
 6.617  + <param name="Name" value="Fisheye Projection">
 6.618 + <param name="Local" value="Fisheye_Projection.html">
 6.619  + <param name="Name" value="Rectilinear Projection">
 6.620 + html">
 6.621  + <param name="Name" value="Stereographic Projection">
 6.622 + <param name="Local" value="Stereographic_Projection.html">
 6.623  + <param name="Name" value="The General Panini Projection">
 6.624 + <param name="Local" value="The_General_Panini_Projection.html">
 6.625  + </OBJECT>
 6.626  + <LI> <OBJECT type="text/sitemap">
 6.627  + <param name="Name" value="PSD">
 6.628  + <param name="Name" value="PSD">
 6.629 + <param name="Local" value="PSD.html">
 6.630  + </OBJECT>
 6.631 + <LI> <OBJECT type="text/sitemap">
 6.632  + <param name="Name" value="PTBlender">
 6.633  + <param name="Name" value="PTblender">
 6.634 + <param name="Local" value="PTblender.html">
 6.635  + </OBJECT>
 6.636  + <LI> <OBJECT type="text/sitemap">
 6.637  + <param name="Name" value="PTMender">
 6.638  + <param name="Name" value="PTmender">
 6.639 + <param name="Local" value="PTmender.html">
 6.640  + </OBJECT>
 6.641  + <LI> <OBJECT type="text/sitemap">
 6.642  + <param name="Name" value="Pto_merge">
 6.643  + <param name="Name" value="Pto merge">
 6.644 + html">
 6.645  + </OBJECT>
 6.646  + <LI> <OBJECT type="text/sitemap">
 6.647  + <param name="Name" value="Pto2mk">
 6.648  + <param name="Name" value="Pto2mk">
 6.649 + <param name="Local" value="Pto2mk.html">
 6.650  + </OBJECT>
 6.651  + <LI> <OBJECT type="text/sitemap">
 6.652  + <param name="Name" value="PTOptimizer">
 6.653  + <param name="Name" value="PTOptimizer">
 6.654 + <param name="Local" value="PTOptimizer.html">
 6.655  + </OBJECT>
 6.656  + <LI> <OBJECT type="text/sitemap">
 6.657  + <param name="Name" value="PTStitcher"> 6.658  + <param name="Name" value="PTStitcher">
 6.659 + <param name="Local" value="PTStitcher.html">
 6.660  + </OBJECT>
 6.661  + <LI> <OBJECT type="text/sitemap">
 6.662  + <param name="Name" value="PTtiff2psd">
 6.663  + <param name="Name" value="PTtiff2psd">
 6.664 + <param name="Local" value="PTtiff2psd.html">
 6.665  + </OBJECT>
 6.666  + <LI> <OBJECT type="text/sitemap">
 6.667  + <param name="Name" value="QTVR">
 6.668  + <param name="Name" value="QTVR">
 6.669 + <param name="Local" value="QTVR.html">
 6.670  + </OBJECT>
 6.671  + <LI> <OBJECT type="text/sitemap"> 6.672  + <param name="Name" value="Qtvr2img">
 6.673  + <param name="Name" value="Qtvr2img">
 6.674 + <param name="Local" value="Qtvr2img.html">
 6.675  + </OBJECT>
 6.676  + <LI> <OBJECT type="text/sitemap">
 6.677  + <param name="Name" value="RAW">
 6.678  + <param name="Name" value="RAW">
 6.679 + <param name="Local" value="RAW.html">
 6.680  + </OBJECT>
 6.681  + <LI> <OBJECT type="text/sitemap">
 6.682  + <param name="Name" value="Rectilinear projection">
 6.683  + <param name="Name" value="Rectilinear Projection">
 6.684 + html">
 6.685  + </OBJECT>
 6.686  + <LI> <OBJECT type="text/sitemap">
 6.687  + <param name="Name" value="Reset values window">
 6.688  + <param name="Name" value="Hugin Reset Values window">
 6.689 + <param name="Local" value="Hugin_Reset_Values_window.html">
 6.690  + </OBJECT>
 6.691  + <LI> <OBJECT type="text/sitemap">
 6.692  + <param name="Name" value="RGBE">
 6.693  + <param name="Name" value="RGBE">
 6.694 + <param name="Local" value="RGBE.html">
 6.695  + </OBJECT>
 6.696  + <LI> <OBJECT type="text/sitemap">
 6.697 + <param name="Name" value="Roll">
 6.698  + <param name="Name" value="Roll">
 6.699 + <param name="Local" value="Roll.html">
 6.700  + </OBJECT>
 6.701  + <LI> <OBJECT type="text/sitemap">
 6.702  + <param name="Name" value="Scripting">
 6.703  + <param name="Name" value="Panorama scripting in a nutshell">
 6.704 + <param name="Local" value="Panorama_scripting_in_a_nutshell.html">
 6.705  + </OBJECT>
 6.706  + <LI> <OBJECT type="text/sitemap">
 6.707  + <param name="Name" value="Smartblend">
 6.708  + <param name="Name" value="SmartBlend">
 6.709 + html">
 6.710  + </OBJECT>
 6.711  + <LI> <OBJECT type="text/sitemap">
 6.712  + <param name="Name" value="Spherical">
 6.713  + <param name="Name" value="Spherical">
 6.714 + <param name="Local" value="Spherical.html">
 6.715  + </OBJECT>
 6.716  + <LI> <OBJECT type="text/sitemap">
 6.717  + <param name="Name" value="Stereographic projection">
 6.718  + <param name="Name" value="Stereographic Projection">
 6.719 + <param name="Local" value="Stereographic_Projection.html">
 6.720  + </OBJECT>
 6.721  + <LI> <OBJECT type="text/sitemap"> 6.722  + <param name="Name" value="Stitcher tab">
 6.723  + <param name="Name" value="Hugin Stitcher tab">
 6.724 + <param name="Local" value="Hugin_Stitcher_tab.html">
 6.725  + </OBJECT>
 6.726  + <LI> <OBJECT type="text/sitemap">
 6.727  + <param name="Name" value="Straight line control point">
 6.728  + <param name="Name" value="Straight line control points">
 6.729 + <param name="Local" value="Straight_line_control_points.html">
 6.730  + </OBJECT>
 6.731  + <LI> <OBJECT type="text/sitemap">
 6.732  + <param name="Name" value="Swing rod">
 6.733 + <param name="Name" value="Swing rod">
 6.734 + <param name="Local" value="Swing_rod.html">
 6.735  + </OBJECT>
 6.736  + <LI> <OBJECT type="text/sitemap">
 6.737  + <param name="Name" value="Tca_correct">
 6.738  + <param name="Name" value="Tca correct">
 6.739 + <param name="Local" value="Tca_correct.html">
 6.740  + </OBJECT>
 6.741  + <LI> <OBJECT type="text/sitemap">
 6.742  + <param name="Name" value="TIFF">
 6.743  + <param name="Name" value="TIFF">
 6.744 + <param name="Local" value="TIFF.html">
 6.745  + </OBJECT>
 6.746  + <LI> <OBJECT type="text/sitemap">
 6.747 + <param name="Name" value="Tone mapping">
 6.748  + <param name="Name" value="Tone mapping">
 6.749 + <param name="Local" value="Tone_mapping.html">
 6.750  + </OBJECT>
 6.751  + <LI> <OBJECT type="text/sitemap">
 6.752  + <param name="Name" value="Trackers">
 6.753  + <param name="Name" value="Hugin Trackers">
 6.754 + <param name="Local" value="Hugin_Trackers.html">
 6.755  + </OBJECT>
 6.756  + <LI> <OBJECT type="text/sitemap">
 6.757  + <param name="Name" value="Translation guide">
 6.758  + <param name="Name" value="Hugin translation guide">
 6.759 + html">
 6.760  + </OBJECT>
 6.761  + <LI> <OBJECT type="text/sitemap">
 6.762  + <param name="Name" value="Vertical control point">
 6.763  + <param name="Name" value="Vertical control points">
 6.764 + <param name="Local" value="Vertical_control_points.html">
 6.765  + </OBJECT>
 6.766  + <LI> <OBJECT type="text/sitemap">
 6.767  + <param name="Name" value="Vig_optimize">
 6.768  + <param name="Name" value="Vig optimize">
 6.769 + <param name="Local" value="Vig_optimize.html">
 6.770  + </OBJECT>
 6.771 + <LI> <OBJECT type="text/sitemap">
 6.772  + <param name="Name" value="Vignetting">
 6.773  + <param name="Name" value="Vignetting">
 6.774 + <param name="Local" value="Vignetting.html">
 6.775  + </OBJECT>
 6.776  + <LI> <OBJECT type="text/sitemap">
 6.777  + <param name="Name" value="Wavy distortion">
 6.778  + <param name="Name" value="Wavy distortion">
 6.779 + <param name="Local" value="Wavy_distortion.html">
 6.780  + </OBJECT>
 6.781  + <LI> <OBJECT type="text/sitemap">
 6.782  + <param name="Name" value="Yaw">
 6.783  + <param name="Name" value="Yaw">
 6.784 + html">
 6.785  + </OBJECT>
 6.786  + <LI> <OBJECT type="text/sitemap">
 6.787  + <param name="Name" value="Zenith">
 6.788  + <param name="Name" value="Zenith">
 6.789 + <param name="Local" value="Zenith.html">
 6.790  + </OBJECT>
 6.791  +</UL>
 6.792  +</BODY></HTML>
     7.1 --- /dev/null	Thu Jan 01 00:00:00 1970 +0000
     7.2 +++ b/src/hugin1/hugin/xrc/data/help_en_EN/help_toc_en.hhc	Mon Nov 08 21:00:19 2010 +0100
     7.3 @@ -0,0 +1,333 @@
     7.4 +<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML//EN">
     7.5 +<HTML>
     7.6 +<HEAD>
     7.7 +<meta name="GENERATOR" content="Microsoft&reg; HTML Help Workshop 4.1">
     7.8 +<!-- Sitemap 1.0 -->
     7.9 +</HEAD><BODY>
    7.10 +<OBJECT type="text/site properties">
    7.11 +	<param name="Window Styles" value="0x800025">
    7.12 +	<param name="ImageType" value="Folder">
    7.13 +</OBJECT>
    7.14 +<UL>
    7.15 +	<LI> <OBJECT type="text/sitemap">
    7.16 +		<param name="Name" value="Hugin">
    7.17 +		<param name="Local" value="Hugin.html">
    7.18 +		</OBJECT>
    7.19 +	<UL>
    7.20 +		<LI> <OBJECT type="text/sitemap">
    7.21 +			<param name="Name" value="Assistant tab">
    7.22 +			<param name="Local" value="Hugin_Assistant_tab.html">
    7.23 +			</OBJECT>
    7.24 +		<LI> <OBJECT type="text/sitemap">
    7.25 +			<param name="Name" value="Images tab">
    7.26 +			<param name="Local" value="Hugin_Images_tab.html">
    7.27 +			</OBJECT>
    7.28 +		<LI> <OBJECT type="text/sitemap">
    7.29 +			<param name="Name" value="Camara and Lens tab">
    7.30 +			<param name="Local" value="Hugin_Camera_and_Lens_tab.html">
    7.31 +			</OBJECT>
    7.32 +		<LI> <OBJECT type="text/sitemap">
    7.33 +			<param name="Name" value="Crop tab">
    7.34 +			<param name="Local" value="Hugin_Crop_tab.html">
    7.35 +			</OBJECT>
    7.36 +		<LI> <OBJECT type="text/sitemap">
    7.37 +			<param name="Name" value="Mask tab">
    7.38 +			<param name="Local" value="Hugin_Mask_tab.html">
    7.39 +			</OBJECT>
    7.40 +		<LI> <OBJECT type="text/sitemap">
    7.41 +			<param name="Name" value="Control points tab">
    7.42 +			<param name="Local" value="Hugin_Control_Points_tab.html">
    7.43 +			</OBJECT>
    7.44 +		<LI> <OBJECT type="text/sitemap">
    7.45 +			<param name="Name" value="Optimizer tab">
    7.46 +			<param name="Local" value="Hugin_Optimizer_tab.html">
    7.47 +			</OBJECT>
    7.48 +		<LI> <OBJECT type="text/sitemap">
    7.49 +			<param name="Name" value="Exposure tab">
    7.50 +			<param name="Local" value="Hugin_Exposure_tab.html">
    7.51 +			</OBJECT>
    7.52 +		<LI> <OBJECT type="text/sitemap">
    7.53 +			<param name="Name" value="Stitcher tab">
    7.54 +			<param name="Local" value="Hugin_Stitcher_tab.html">
    7.55 +			</OBJECT>
    7.56 +		<LI> <OBJECT type="text/sitemap">
    7.57 +			<param name="Name" value="Batch processor">
    7.58 +			<param name="Local" value="Hugin_Batch_Processor.html">
    7.59 +			</OBJECT>
    7.60 +		<LI> <OBJECT type="text/sitemap">
    7.61 +			<param name="Name" value="Preferences">
    7.62 +			<param name="Local" value="Hugin_Preferences.html">
    7.63 +			</OBJECT>
    7.64 +		<LI> <OBJECT type="text/sitemap">
    7.65 +			<param name="Name" value="Preview window">
    7.66 +			<param name="Local" value="Hugin_Preview_window.html">
    7.67 +			</OBJECT>
    7.68 +		<LI> <OBJECT type="text/sitemap">
    7.69 +			<param name="Name" value="Fast preview window">
    7.70 +			<param name="Local" value="Hugin_Fast_Preview_window.html">
    7.71 +			</OBJECT>
    7.72 +		<LI> <OBJECT type="text/sitemap">
    7.73 +			<param name="Name" value="Control points table">
    7.74 +			<param name="Local" value="Hugin_Control_Points_table.html">
    7.75 +			</OBJECT>
    7.76 +		<LI> <OBJECT type="text/sitemap">
    7.77 +			<param name="Name" value="Reset values window">
    7.78 +			<param name="Local" value="Hugin_Reset_Values_window.html">
    7.79 +			</OBJECT>
    7.80 +		<LI> <OBJECT type="text/sitemap">
    7.81 +			<param name="Name" value="Keyboard shortcuts">
    7.82 +			<param name="Local" value="Hugin_Keyboard_shortcuts.html">
    7.83 +			</OBJECT>
    7.84 +		<LI> <OBJECT type="text/sitemap">
    7.85 +			<param name="Name" value="Frequently asked questions">
    7.86 +			<param name="Local" value="Hugin_FAQ.html">
    7.87 +			</OBJECT>
    7.88 +	</UL>
    7.89 +	<LI> <OBJECT type="text/sitemap">
    7.90 +		<param name="Name" value="Command line tools">
    7.91 +		</OBJECT>
    7.92 +	<UL>
    7.93 +		<LI> <OBJECT type="text/sitemap">
    7.94 +			<param name="Name" value="Nona">
    7.95 +			<param name="Local" value="Nona.html">
    7.96 +			</OBJECT>
    7.97 +		<LI> <OBJECT type="text/sitemap">
    7.98 +			<param name="Name" value="Fulla">
    7.99 +			<param name="Local" value="Fulla.html">
   7.100 +			</OBJECT>
   7.101 +		<LI> <OBJECT type="text/sitemap">
   7.102 +			<param name="Name" value="Nona GUI">
   7.103 +			<param name="Local" value="Nona_gui.html">
   7.104 +			</OBJECT>
   7.105 +		<LI> <OBJECT type="text/sitemap">
   7.106 +			<param name="Name" value="Autooptimiser">
   7.107 +			<param name="Local" value="Autooptimiser.html">
   7.108 +			</OBJECT>
   7.109 +		<LI> <OBJECT type="text/sitemap">
   7.110 +			<param name="Name" value="Align_image_stack">
   7.111 +			<param name="Local" value="Align_image_stack.html">
   7.112 +			</OBJECT>
   7.113 +		<LI> <OBJECT type="text/sitemap">
   7.114 +			<param name="Name" value="Tca_correct">
   7.115 +			<param name="Local" value="Tca_correct.html">
   7.116 +			</OBJECT>
   7.117 +		<LI> <OBJECT type="text/sitemap">
   7.118 +			<param name="Name" value="Matchpoint">
   7.119 +			<param name="Local" value="MatchPoint.html">
   7.120 +			</OBJECT>
   7.121 +		<LI> <OBJECT type="text/sitemap">
   7.122 +			<param name="Name" value="Pto2mk">
   7.123 +			<param name="Local" value="Pto2mk.html">
   7.124 +			</OBJECT>
   7.125 +		<LI> <OBJECT type="text/sitemap">
   7.126 +			<param name="Name" value="Cpclean">
   7.127 +			<param name="Local" value="Cpclean.html">
   7.128 +			</OBJECT>
   7.129 +		<LI> <OBJECT type="text/sitemap">
   7.130 +			<param name="Name" value="Cpfind">
   7.131 +			<param name="Local" value="Cpfind.html">
   7.132 +			</OBJECT>
   7.133 +		<LI> <OBJECT type="text/sitemap">
   7.134 +			<param name="Name" value="Icpfind">
   7.135 +			<param name="Local" value="Icpfind.html">
   7.136 +			</OBJECT>
   7.137 +		<LI> <OBJECT type="text/sitemap">
   7.138 +			<param name="Name" value="Vig_optimise">
   7.139 +			<param name="Local" value="Vig_optimize.html">
   7.140 +			</OBJECT>
   7.141 +		<LI> <OBJECT type="text/sitemap">
   7.142 +			<param name="Name" value="Celeste_standalone">
   7.143 +			<param name="Local" value="Celeste_standalone.html">
   7.144 +			</OBJECT>
   7.145 +		<LI> <OBJECT type="text/sitemap">
   7.146 +			<param name="Name" value="PTBatcher">
   7.147 +			<param name="Local" value="Hugin_Batch_Processor.html">
   7.148 +			</OBJECT>
   7.149 +		<LI> <OBJECT type="text/sitemap">
   7.150 +			<param name="Name" value="PTBatcherGUI">
   7.151 +			<param name="Local" value="Hugin_Batch_Processor.html">
   7.152 +			</OBJECT>
   7.153 +		<LI> <OBJECT type="text/sitemap">
   7.154 +			<param name="Name" value="Pano_modify">
   7.155 +			<param name="Local" value="Pano_modify.html">
   7.156 +			</OBJECT>
   7.157 +		<LI> <OBJECT type="text/sitemap">
   7.158 +			<param name="Name" value="Pto_merge">
   7.159 +			<param name="Local" value="Pto_merge.html">
   7.160 +			</OBJECT>
   7.161 +		<LI> <OBJECT type="text/sitemap">
   7.162 +			<param name="Name" value="External programs">
   7.163 +			</OBJECT>
   7.164 +		<UL>
   7.165 +			<LI> <OBJECT type="text/sitemap">
   7.166 +				<param name="Name" value="Enblend/Enfuse">
   7.167 +				</OBJECT>
   7.168 +			<UL>
   7.169 +				<LI> <OBJECT type="text/sitemap">
   7.170 +					<param name="Name" value="Enblend">
   7.171 +					<param name="Local" value="Enblend.html">
   7.172 +					</OBJECT>
   7.173 +				<LI> <OBJECT type="text/sitemap">
   7.174 +					<param name="Name" value="Enblend manual">
   7.175 +					<param name="Local" value="Enblend_reference_manual.html">
   7.176 +					</OBJECT>
   7.177 +				<LI> <OBJECT type="text/sitemap">
   7.178 +					<param name="Name" value="Enfuse">
   7.179 +					<param name="Local" value="Enfuse.html">
   7.180 +					</OBJECT>
   7.181 +				<LI> <OBJECT type="text/sitemap">
   7.182 +					<param name="Name" value="Enfuse manual">
   7.183 +					<param name="Local" value="Enfuse_reference_manual.html">
   7.184 +					</OBJECT>
   7.185 +			</UL>
   7.186 +			<LI> <OBJECT type="text/sitemap">
   7.187 +				<param name="Name" value="Panorama Tools">
   7.188 +				<param name="Local" value="Panorama_tools.html">
   7.189 +				</OBJECT>
   7.190 +			<UL>
   7.191 +				<LI> <OBJECT type="text/sitemap">
   7.192 +					<param name="Name" value="PTBlender">
   7.193 +					<param name="Local" value="PTblender.html">
   7.194 +					</OBJECT>
   7.195 +				<LI> <OBJECT type="text/sitemap">
   7.196 +					<param name="Name" value="PTMender">
   7.197 +					<param name="Local" value="PTmender.html">
   7.198 +					</OBJECT>
   7.199 +				<LI> <OBJECT type="text/sitemap">
   7.200 +					<param name="Name" value="PTOptimizer">
   7.201 +					<param name="Local" value="PTOptimizer.html">
   7.202 +					</OBJECT>
   7.203 +				<LI> <OBJECT type="text/sitemap">
   7.204 +					<param name="Name" value="PTStitcher">
   7.205 +					<param name="Local" value="PTStitcher.html">
   7.206 +					</OBJECT>
   7.207 +			</UL>
   7.208 +			<LI> <OBJECT type="text/sitemap">
   7.209 +				<param name="Name" value="Smartblend">
   7.210 +				<param name="Local" value="SmartBlend.html">
   7.211 +				</OBJECT>
   7.212 +		</UL>
   7.213 +	</UL>
   7.214 +	<LI> <OBJECT type="text/sitemap">
   7.215 +		<param name="Name" value="Control points detectors">
   7.216 +		</OBJECT>
   7.217 +	<UL>
   7.218 +		<LI> <OBJECT type="text/sitemap">
   7.219 +			<param name="Name" value="Overview">
   7.220 +			<param name="Local" value="Control_point_generators.html">
   7.221 +			</OBJECT>
   7.222 +		<LI> <OBJECT type="text/sitemap">
   7.223 +			<param name="Name" value="Typical parameters">
   7.224 +			<param name="Local" value="Control_Point_Detector_Parameters.html">
   7.225 +			</OBJECT>
   7.226 +		<LI> <OBJECT type="text/sitemap">
   7.227 +			<param name="Name" value="Setup of control point detectors">
   7.228 +			<param name="Local" value="Hugin_Parameters_for_Control_Point_Detectors_dialog.html">
   7.229 +			</OBJECT>
   7.230 +		<LI> <OBJECT type="text/sitemap">
   7.231 +			<param name="Name" value="Cpfind">
   7.232 +			<param name="Local" value="Cpfind.html">
   7.233 +			</OBJECT>
   7.234 +		<LI> <OBJECT type="text/sitemap">
   7.235 +			<param name="Name" value="Autopano">
   7.236 +			<param name="Local" value="Autopano.html">
   7.237 +			</OBJECT>
   7.238 +		<LI> <OBJECT type="text/sitemap">
   7.239 +			<param name="Name" value="Autopano-SIFT">
   7.240 +			<param name="Local" value="Autopano-sift.html">
   7.241 +			</OBJECT>
   7.242 +		<LI> <OBJECT type="text/sitemap">
   7.243 +			<param name="Name" value="Autopano-SIFT-C">
   7.244 +			<param name="Local" value="Autopano-sift-C.html">
   7.245 +			</OBJECT>
   7.246 +		<LI> <OBJECT type="text/sitemap">
   7.247 +			<param name="Name" value="Panomatic">
   7.248 +			<param name="Local" value="Panomatic.html">
   7.249 +			</OBJECT>
   7.250 +		<LI> <OBJECT type="text/sitemap">
   7.251 +			<param name="Name" value="Align_image_stack">
   7.252 +			<param name="Local" value="Align_image_stack.html">
   7.253 +			</OBJECT>
   7.254 +	</UL>
   7.255 +	<LI> <OBJECT type="text/sitemap">
   7.256 +		<param name="Name" value="Tips and Tricks">
   7.257 +		</OBJECT>
   7.258 +	<UL>
   7.259 +		<LI> <OBJECT type="text/sitemap">
   7.260 +			<param name="Name" value="Align a stack of photos">
   7.261 +			<param name="Local" value="Align_a_stack_of_photos.html">
   7.262 +			</OBJECT>
   7.263 +		<LI> <OBJECT type="text/sitemap">
   7.264 +			<param name="Name" value="16 bit workflow with Hugin">
   7.265 +			<param name="Local" value="16bit_workflow_with_hugin.html">
   7.266 +			</OBJECT>
   7.267 +		<LI> <OBJECT type="text/sitemap">
   7.268 +			<param name="Name" value="Using celeste with hugin">
   7.269 +			<param name="Local" value="Using_Celeste_with_hugin.html">
   7.270 +			</OBJECT>
   7.271 +		<LI> <OBJECT type="text/sitemap">
   7.272 +			<param name="Name" value="HDR workflow with hugin">
   7.273 +			<param name="Local" value="HDR_workflow_with_hugin.html">
   7.274 +			</OBJECT>
   7.275 +		<LI> <OBJECT type="text/sitemap">
   7.276 +			<param name="Name" value="Scripting">
   7.277 +			<param name="Local" value="Panorama_scripting_in_a_nutshell.html">
   7.278 +			</OBJECT>
   7.279 +	</UL>
   7.280 +	<LI> <OBJECT type="text/sitemap">
   7.281 +		<param name="Name" value="General">
   7.282 +		</OBJECT>
   7.283 +	<UL>
   7.284 +		<LI> <OBJECT type="text/sitemap">
   7.285 +			<param name="Name" value="Panorama">
   7.286 +			<param name="Local" value="Panorama.html">
   7.287 +			</OBJECT>
   7.288 +		<LI> <OBJECT type="text/sitemap">
   7.289 +			<param name="Name" value="Panorama formats">
   7.290 +			<param name="Local" value="Panorama_formats.html">
   7.291 +			</OBJECT>
   7.292 +		<LI> <OBJECT type="text/sitemap">
   7.293 +			<param name="Name" value="Projections">
   7.294 +			<param name="Local" value="Projections.html">
   7.295 +			</OBJECT>
   7.296 +		<UL>
   7.297 +			<LI> <OBJECT type="text/sitemap">
   7.298 +				<param name="Name" value="Rectilinear projection">
   7.299 +				<param name="Local" value="Rectilinear_Projection.html">
   7.300 +				</OBJECT>
   7.301 +			<LI> <OBJECT type="text/sitemap">
   7.302 +				<param name="Name" value="Equirectangular projection">
   7.303 +				<param name="Local" value="Equirectangular_Projection.html">
   7.304 +				</OBJECT>
   7.305 +			<LI> <OBJECT type="text/sitemap">
   7.306 +				<param name="Name" value="Cylindrical projection">
   7.307 +				<param name="Local" value="Cylindrical_Projection.html">
   7.308 +				</OBJECT>
   7.309 +			<LI> <OBJECT type="text/sitemap">
   7.310 +				<param name="Name" value="Cubic projection">
   7.311 +				<param name="Local" value="Cubic_Projection.html">
   7.312 +				</OBJECT>
   7.313 +			<LI> <OBJECT type="text/sitemap">
   7.314 +				<param name="Name" value="Fisheye projection">
   7.315 +				<param name="Local" value="Fisheye_Projection.html">
   7.316 +				</OBJECT>
   7.317 +			<LI> <OBJECT type="text/sitemap">
   7.318 +				<param name="Name" value="Stereographic projection">
   7.319 +				<param name="Local" value="Stereographic_Projection.html">
   7.320 +				</OBJECT>
   7.321 +			<LI> <OBJECT type="text/sitemap">
   7.322 +				<param name="Name" value="General Panini projection">
   7.323 +				<param name="Local" value="The_General_Panini_Projection.html">
   7.324 +				</OBJECT>
   7.325 +		</UL>
   7.326 +		<LI> <OBJECT type="text/sitemap">
   7.327 +			<param name="Name" value="Panorama tools">
   7.328 +			<param name="Local" value="Panorama_tools.html">
   7.329 +			</OBJECT>
   7.330 +	</UL>
   7.331 +	<LI> <OBJECT type="text/sitemap">
   7.332 +		<param name="Name" value="Licence">
   7.333 +		<param name="Local" value="license.html">
   7.334 +		</OBJECT>
   7.335 +</UL>
   7.336 +</BODY></HTML>
     8.1 --- /dev/null	Thu Jan 01 00:00:00 1970 +0000
     8.2 +++ b/src/hugin1/hugin/xrc/data/help_en_EN/hhc.cmake	Mon Nov 08 21:00:19 2010 +0100
     8.3 @@ -0,0 +1,3 @@
     8.4 +# html help compiler is returning non zero retun value
     8.5 +# so using execute_process to discard the return value
     8.6 +EXECUTE_PROCESS(COMMAND ${HTML_HELP_COMPILER} hugin_help_en_EN.hhp ERROR_QUIET)
     8.7 \ No newline at end of file
     9.1 --- /dev/null	Thu Jan 01 00:00:00 1970 +0000
     9.2 +++ b/src/hugin1/hugin/xrc/data/help_en_EN/hugin_help_en_EN.hhp	Mon Nov 08 21:00:19 2010 +0100
     9.3 @@ -0,0 +1,155 @@
     9.4 +[OPTIONS]
     9.5 +Compatibility=1.1 or later
     9.6 +Compiled file=hugin_help_en_EN.chm
     9.7 +Contents file=help_toc_en.hhc
     9.8 +Default topic=Hugin.html
     9.9 +Display compile progress=Yes
    9.10 +Flat=Yes
    9.11 +Full-text search=Yes
    9.12 +Index file=help_index_en.hhk
    9.13 +Language=0x809 Englisch (Gro�britannien)
    9.14 +Title=Hugin Help
    9.15 +
    9.16 +
    9.17 +[FILES]
    9.18 +16bit.html
    9.19 +16bit_workflow_with_hugin.html
    9.20 +Aliasing.html
    9.21 +Align_a_stack_of_photos.html
    9.22 +Align_image_stack.html
    9.23 +Alpha_channel.html
    9.24 +Aspect_Ratio.html
    9.25 +Autooptimiser.html
    9.26 +Autopano.html
    9.27 +Autopano-sift.html
    9.28 +Autopano-sift-C.html
    9.29 +Banding.html
    9.30 +Barrel_distortion.html
    9.31 +Bracketing.html
    9.32 +Camera_response_curve.html
    9.33 +CCD.html
    9.34 +Celeste_standalone.html
    9.35 +Chromatic_aberration.html
    9.36 +Color_correct_tiff.html
    9.37 +Colour_profile.html
    9.38 +Contrast.html
    9.39 +Control_Point_Detector_Parameters.html
    9.40 +Control_point_generators.html
    9.41 +Control_points.html
    9.42 +Cpclean.html
    9.43 +Cpfind.html
    9.44 +Crop_factor.html
    9.45 +Cropped_TIFF.html
    9.46 +Cubic_Projection.html
    9.47 +Cylindrical_panorama.html
    9.48 +Cylindrical_Projection.html
    9.49 +Depth_of_Field.html
    9.50 +DSLR_spherical_resolution.html
    9.51 +Dust_Removal_with_Flatfield.html
    9.52 +Dynamic_range.html
    9.53 +Enblend.html
    9.54 +Enblend_reference_manual.html
    9.55 +Enfuse.html
    9.56 +Enfuse_reference_manual.html
    9.57 +Equirectangular_Projection.html
    9.58 +EXIF.html
    9.59 +Field_of_View.html
    9.60 +Fisheye_Projection.html
    9.61 +Focal_Length.html
    9.62 +Freepv.html
    9.63 +Fulla.html
    9.64 +Gamma.html
    9.65 +GIF.html
    9.66 +HDR.html
    9.67 +HDR_workflow_with_hugin.html
    9.68 +Horizontal_control_points.html
    9.69 +Hugin.html
    9.70 +Hugin_Assistant_tab.html
    9.71 +Hugin_Batch_Processor.html
    9.72 +Hugin_Camera_and_Lens_tab.html
    9.73 +Hugin_Control_Points_tab.html
    9.74 +Hugin_Control_Points_table.html
    9.75 +Hugin_Crop_tab.html
    9.76 +Hugin_Exposure_tab.html
    9.77 +Hugin_FAQ.html
    9.78 +Hugin_Fast_Preview_window.html
    9.79 +Hugin_Images_tab.html
    9.80 +Hugin_Keyboard_shortcuts.html
    9.81 +Hugin_Main_window.html
    9.82 +Hugin_Mask_tab.html
    9.83 +Hugin_Optimizer_tab.html
    9.84 +Hugin_Parameters_for_Control_Point_Detectors_dialog.html
    9.85 +Hugin_Preferences.html
    9.86 +Hugin_Preview_window.html
    9.87 +Hugin_Reset_Values_window.html
    9.88 +Hugin_Stitcher_tab.html
    9.89 +Hugin_Trackers.html
    9.90 +Hugin_translation_guide.html
    9.91 +Icpfind.html
    9.92 +Interpolation.html
    9.93 +JPEG.html
    9.94 +Landscape.html
    9.95 +Lens_correction_model.html
    9.96 +Lens_distortion.html
    9.97 +license.html
    9.98 +Lightprobe.html
    9.99 +MatchPoint.html
   9.100 +Nadir.html
   9.101 +Nodal_Point.html
   9.102 +Nona.html
   9.103 +Nona_gui.html
   9.104 +No-parallax_point.html
   9.105 +OpenEXR.html
   9.106 +Optimization.html
   9.107 +Pano12.html
   9.108 +Pano_modify.html
   9.109 +Panoglview.html
   9.110 +Panoinfo.html
   9.111 +Panomatic.html
   9.112 +Panorama.html
   9.113 +Panorama_formats.html
   9.114 +Panorama_scripting_in_a_nutshell.html
   9.115 +Panorama_tools.html
   9.116 +Parallax.html
   9.117 +Perspective_correction.html
   9.118 +Perspective_distortion.html
   9.119 +Pfstmo.html
   9.120 +Pincushion_distortion.html
   9.121 +Pitch.html
   9.122 +PNG.html
   9.123 +Portrait.html
   9.124 +PPI.html
   9.125 +Projections.html
   9.126 +PSD.html
   9.127 +PTblender.html
   9.128 +PTmender.html
   9.129 +Pto2mk.html
   9.130 +Pto_merge.html
   9.131 +PTOptimizer.html
   9.132 +PTStitcher.html
   9.133 +PTtiff2psd.html
   9.134 +QTVR.html
   9.135 +Qtvr2img.html
   9.136 +RAW.html
   9.137 +Rectilinear_Projection.html
   9.138 +RGBE.html
   9.139 +Roll.html
   9.140 +SmartBlend.html
   9.141 +Spherical.html
   9.142 +Stereographic_Projection.html
   9.143 +Straight_line_control_points.html
   9.144 +Swing_rod.html
   9.145 +Tca_correct.html
   9.146 +The_General_Panini_Projection.html
   9.147 +TIFF.html
   9.148 +Tone_mapping.html
   9.149 +Using_Celeste_with_hugin.html
   9.150 +Vertical_control_points.html
   9.151 +Vig_optimize.html
   9.152 +Vignetting.html
   9.153 +Wavy_distortion.html
   9.154 +Yaw.html
   9.155 +Zenith.html
   9.156 +
   9.157 +[INFOTYPES]
   9.158 +
    10.1 --- a/src/hugin1/hugin/xrc/data/help_it_IT/CMakeLists.txt	Mon Nov 08 15:14:51 2010 +0000
    10.2 +++ b/src/hugin1/hugin/xrc/data/help_it_IT/CMakeLists.txt	Mon Nov 08 21:00:19 2010 +0100
    10.3 @@ -1,6 +1,13 @@
    10.4 +
    10.5 +IF(WIN32)
    10.6 +
    10.7 +#TODO generate italian version of help file
    10.8 +
    10.9 +ELSE(WIN32)
   10.10  
   10.11  FILE(GLOB DATAFILES *.htm *.html *.ico *.gif *.JPG *.jpg *.png *.xpm *.hhc *.hhk *.hhp
   10.12  *.manual)
   10.13  
   10.14  INSTALL(FILES ${DATAFILES} DESTINATION ${HUGINDATADIR}/xrc/data/help_it_IT)
   10.15  
   10.16 +ENDIF(WIN32)
   10.17 \ No newline at end of file
    11.1 --- a/src/hugin1/ptbatcher/BatchFrame.cpp	Mon Nov 08 15:14:51 2010 +0000
    11.2 +++ b/src/hugin1/ptbatcher/BatchFrame.cpp	Mon Nov 08 21:00:19 2010 +0100
    11.3 @@ -111,7 +111,9 @@
    11.4  	m_cancelled = false;
    11.5  	m_closeThread = false;
    11.6  	//m_paused = false;
    11.7 +#ifndef __WXMSW__
    11.8  	m_help=0;
    11.9 +#endif
   11.10  
   11.11  	//load xrc resources
   11.12  	wxXmlResource::Get()->LoadFrame(this, (wxWindow* )NULL, wxT("batch_frame"));
   11.13 @@ -492,6 +494,9 @@
   11.14  void BatchFrame::OnButtonHelp(wxCommandEvent &event)
   11.15  {
   11.16  	DEBUG_TRACE("");
   11.17 +#ifdef __WXMSW__
   11.18 +    GetHelpController().DisplaySection(wxT("Hugin_Batch_Processor.html"));
   11.19 +#else
   11.20      if (m_help == 0)
   11.21      {
   11.22  #if defined __WXMAC__ && defined MAC_SELF_CONTAINED_BUNDLE
   11.23 @@ -531,6 +536,7 @@
   11.24      }
   11.25      m_help->Display(wxT("Hugin_Batch_Processor.html"));
   11.26  	//DisplayHelp(wxT("Hugin_Batch_Processor.html"));
   11.27 +#endif
   11.28  }
   11.29  void BatchFrame::OnButtonMoveDown(wxCommandEvent &event)
   11.30  {
   11.31 @@ -947,7 +953,9 @@
   11.32  	m_closeThread = true;
   11.33  	this->GetThread()->Wait();
   11.34  	//wxMessageBox(_T("Closing frame..."));
   11.35 -	delete m_help;
   11.36 +#ifndef __WXMSW__
   11.37 +    delete m_help;
   11.38 +#endif
   11.39  	this->Destroy();
   11.40  }
   11.41  
    12.1 --- a/src/hugin1/ptbatcher/BatchFrame.h	Mon Nov 08 15:14:51 2010 +0000
    12.2 +++ b/src/hugin1/ptbatcher/BatchFrame.h	Mon Nov 08 21:00:19 2010 +0100
    12.3 @@ -31,6 +31,9 @@
    12.4  #include "Batch.h"
    12.5  #include "ProjectListBox.h"
    12.6  #include "DirTraverser.h"
    12.7 +#ifdef __WXMSW__
    12.8 +#include "wx/msw/helpchm.h"
    12.9 +#endif
   12.10  //#include <wx/app.h>
   12.11  
   12.12  /** Simple class that forward the drop to the mainframe */
   12.13 @@ -110,6 +113,11 @@
   12.14  	void AddDirToList(wxString aDir);
   12.15  	void ChangePrefix(int index,wxString newPrefix);
   12.16  
   12.17 +#ifdef __WXMSW__
   12.18 +    /** return help controller for open help */
   12.19 +    wxCHMHelpController& GetHelpController() { return m_msHtmlHelp; }
   12.20 +#endif
   12.21 +
   12.22  	//wxMutex* projListMutex;
   12.23  	ProjectListBox *projListBox;
   12.24  
   12.25 @@ -122,7 +130,11 @@
   12.26  	bool m_closeThread; //included to signal the thread to finish execution
   12.27  	//TO-DO: include a batch or project progress gauge? Test initialization commented out in constructor
   12.28  	//wxGauge* m_gauge;
   12.29 -	wxHtmlHelpController * m_help;
   12.30 +#ifdef __WXMSW__
   12.31 +    wxCHMHelpController m_msHtmlHelp;
   12.32 +#else
   12.33 +    wxHtmlHelpController * m_help;
   12.34 +#endif
   12.35  
   12.36  	void OnProcessTerminate(wxProcessEvent & event);
   12.37  	/** called by thread when queue was changed outside of PTBatcherGUI
    13.1 --- a/src/hugin1/ptbatcher/PTBatcherGUI.cpp	Mon Nov 08 15:14:51 2010 +0000
    13.2 +++ b/src/hugin1/ptbatcher/PTBatcherGUI.cpp	Mon Nov 08 21:00:19 2010 +0100
    13.3 @@ -25,6 +25,9 @@
    13.4   */
    13.5  
    13.6  #include "PTBatcherGUI.h"
    13.7 +#ifdef __WXMSW__
    13.8 +#include "wx/cshelp.h"
    13.9 +#endif
   13.10  
   13.11  // make wxwindows use this class as the main application
   13.12  IMPLEMENT_APP(PTBatcherGUI)
   13.13 @@ -42,6 +45,10 @@
   13.14  #if defined __WXMSW__
   13.15  	int localeID = wxConfigBase::Get()->Read(wxT("language"), (long) wxLANGUAGE_DEFAULT);
   13.16  	m_locale.Init(localeID);
   13.17 +    // initialize help provider
   13.18 +    wxHelpControllerHelpProvider* provider = new wxHelpControllerHelpProvider;
   13.19 +    wxHelpProvider::Set(provider);
   13.20 +
   13.21  #else
   13.22      m_locale.Init(wxLANGUAGE_DEFAULT);
   13.23  #endif
   13.24 @@ -165,6 +172,10 @@
   13.25  	{
   13.26  		m_frame = new BatchFrame(&m_locale,m_xrcPrefix);
   13.27  		m_frame->RestoreSize();
   13.28 +#ifdef __WXMSW__
   13.29 +        provider->SetHelpController(&m_frame->GetHelpController());
   13.30 +        m_frame->GetHelpController().Initialize(m_xrcPrefix+wxT("data/hugin_help_en_EN.chm"));
   13.31 +#endif
   13.32  		SetTopWindow(m_frame);
   13.33  		m_frame->Show(true);
   13.34  		m_server = new BatchIPCServer();

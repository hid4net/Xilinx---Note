

<h1 style="text-align:center">TCL@Vivado</h1>

--------------------------------------------------------------------------------
> 相关文档: Vivado Design Suite Tcl Command Reference Guide, UG835


--------------------------------------------------------------------------------
# 1. 常用的命令

## 1.1. 获取对象
![Vivado-TCL-objects](/assets/Vivado-TCL-objects.png)
`get_cells`, `get_nets`, `get_ports`, `get_pins`, `get_clocks`
`get_selected_objects`

## 1.2. 时序
`get_timing_paths`
`report_timing`
`report_timing_summary`

## 1.3. IO

--------------------------------------------------------------------------------
# 2. 模板

## 2.1. 保存 project
```tcl
## =================================================================================================
##              保存 vivado project 基本信息
## -------------------------------------------------------------------------------------------------
set _fp_ [open ../TCL/xvprj_info w]
puts $_fp_ "prj_name=[get_property NAME [current_project]]"
puts $_fp_ "fpga_part=[get_property PART [current_project]]"
set _tmp_ [get_property BOARD_PART [current_project]]
if {[llength $_tmp_] == 0} {
    set _tmp_ Null
}
puts $_fp_ "board_part=$_tmp_"
puts $_fp_ "top_module=[get_property TOP [current_fileset]]"
close $_fp_

## =================================================================================================
##              导出 IP
## -------------------------------------------------------------------------------------------------
set _ips_ [get_ips -exclude_bd_ips]
if [llength $_ips_] {
    write_ip_tcl -force -no_ip_version $_ips_ ./../TCL/ips.tcl
} else {
    puts "There are no IPs to export!"
}

## =================================================================================================
##              导出 BDs
## -------------------------------------------------------------------------------------------------
foreach _bd_ [get_files -filter {FILE_TYPE=="Block Designs"} -of_objects [current_fileset]] {
    if {[lsearch [list_property $_bd_]  PARENT_COMPOSITE_FILE] == -1} {
        set _bd_name_ [open_bd_design $_bd_]
        write_bd_tcl -force ../TCL/bd_${_bd_name_}.tcl
    }
}
```
## 2.2. 创建/重建 project
```tcl
## =================================================================================================
##              全局变量
## -------------------------------------------------------------------------------------------------
## ---------------- vivado project 参数 ----------------
#* >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> 说明 >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
#* 新建 project:
#*      按需设置 4 个参数, 如果不设置, 请将设置参数的语句注释掉
#*      如果不设置 project 名称, 则以默认参数创建名为 tmp 的 project
#* 重建 project:
#*      请确保已在 vivado 中运行过 `save_project.tcl` 脚本, 并将设置参数的语句注释掉
#*      如果想更改 project 名称, 设置 _prj_name_
#* <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 说明 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
## todo: 设置项目名、FPGA_型号、开发板型号、顶层模块
# set _prj_name_ 项目名
# set _fpga_part_ FPGA_型号
# set _board_part_ 开发板型号
# set _top_module_ 顶层模块名

## ---------------- IPs 和 BDs 脚本 ----------------
#* 如果运行过 `save_project.tcl` 脚本, 可不设置
## todo: 设置 IP 和 BD 脚本路径
# set _ips_tcl_ ips.tcl
# set _bd_tcl_ BD_脚本名

## ---------------- vivado project 路径 ----------------
set _proj_dir_ ./../vivado_project

## =================================================================================================
##              创建/重建 vivado project
## -------------------------------------------------------------------------------------------------
## ---------------- 检查目录是否存在 ----------------
if {! [file exists $_proj_dir_]} {
    file mkdir $_proj_dir_
}
## ---------------- 读取保存的 project 信息 ----------------
if {[file exists xvprj_info]} {
    set _save_prj_info_ ""
    set _fp_ [open xvprj_info]
    foreach line [read $_fp_] {
        append _save_prj_info_ [split $line =]
        append _save_prj_info_ " "
    }
    close $_fp_
}
## ---------------- 创建/重建 project ----------------
## 创建 project
if {[info exists _prj_name_]} {
    create_project $_prj_name_ $_proj_dir_ -force
} elseif {[info exists _save_prj_info_]} {
    create_project [dict get $_save_prj_info_ prj_name] $_proj_dir_ -force
} else {
    create_project tmp $_proj_dir_ -force
}
## 设置 FPGA 型号
if {[info exists _fpga_part_]} {
    set_property part $_fpga_part_ [current_project]
} elseif {[info exists _save_prj_info_]} {
    set_property part [dict get $_save_prj_info_ fpga_part] [current_project]
}
## 设置开发板参数
if {[info exists _board_part_] && [llength $_board_part_]} {
    set_property board_part $_board_part_ [current_project]
    reset_property board_connections [current_project]
} elseif {[info exists _save_prj_info_]} {
    set _board_part_ [dict get $_save_prj_info_ board_part]
    if {![string equal -nocase $_board_part_ Null]} {
        set_property board_part $_board_part_ [current_project]
        reset_property board_connections [current_project]
    }
}

## =================================================================================================
##              添加 RTL 文件
## -------------------------------------------------------------------------------------------------
if {[string equal [get_filesets -quiet sources_1] ""]} {
    create_fileset -srcset sources_1
}
set obj [get_filesets sources_1]

## ---------------- HDL ----------------
## 方法: `add_files` 添加文件夹或文件列表, 可以不指定 fileset, -norecurse 表示不遍历文件夹
## todo: 请确保所有 HDL 都被添加
add_files                               ./../RTL
# add_files -fileset $obj                 ./../RTL
# add_files -fileset $obj -norecurse      ./../RTL
# add_files -fileset $obj -norecurse      [list ...]

## ---------------- IP ----------------
## 方法 1: `add_files` 添加 xci 文件列表
#* 方法 2: `source` IP脚本, 推荐!
## todo: 请确保所有 IP 都被添加
#* 自动导入 IPs
# source $_ips_tcl_
if {[file exists ips.tcl]} {
    source ips.tcl
}

## ---------------- Block Design ----------------
## 方法 1: `add_files` BD文件
#* 方法 2: `source` BD脚本
## todo: 如果有 BD, 请导入
# source $_bd_tcl_
# add_files -fileset sources_1 -norecurse \
#     [make_wrapper -files [get_files -filter [expr "\{NAME=~*/${design_name}.bd}"]] -top]
#* 自动导入 BDs
foreach _bd_tcl_ [glob -nocomplain bd_*.tcl] {
    source $_bd_tcl_
    add_files -fileset $obj -norecurse [make_wrapper -files [get_files ${design_name}.bd] -top]
}

## ---------------- 设置顶层文件 ----------------
if {[info exists _top_module_]} {
    set_property top $_top_module_ $obj
} elseif {[info exists _save_prj_info_]} {
    set_property top [dict get $_save_prj_info_ top_module] $obj
}

update_compile_order -fileset $obj

## =================================================================================================
##              添加 XDC
## -------------------------------------------------------------------------------------------------
if {[string equal [get_filesets -quiet constrs_1] ""]} {
    create_fileset -constrset constrs_1
}
set obj [get_filesets constrs_1]

## todo: 添加 xdc
add_files -fileset $obj             ./../XDC
## todo: 设置目标 xdc
set_property target_constrs_file    ./../XDC/debug.xdc      $obj

## =================================================================================================
##              添加 utils
## -------------------------------------------------------------------------------------------------

## =================================================================================================
##              启动 GUI
## -------------------------------------------------------------------------------------------------
# start_gui

## =================================================================================================
##              启动综合和后端
## -------------------------------------------------------------------------------------------------
# launch_runs synth_1 -jobs [expr $env(NUMBER_OF_PROCESSORS)/2]
# launch_runs impl_1 -jobs [expr $env(NUMBER_OF_PROCESSORS)/2]
# launch_runs impl_1 -to_step write_bitstream -jobs [expr $env(NUMBER_OF_PROCESSORS)/2]
```

* 创建/重建 project 的 bat
    ``` batch
    @echo off
    cd /D %~dp0
    @REM vivado -nojournal -nolog -mode tcl
    vivado -nojournal -nolog -mode batch -source build_project.tcl
    ```

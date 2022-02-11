
Building on Unix
----------------

1) run ./configure to generate build scripts
   Note: type ./configure --help for a list of fine-tuning options

2) type "make"

3) type "make check" to perform self-tests

4) type "make install" to install


Building on Windows
-------------------

If you have run the VC++ VCVARS32.BAT, you should be able to type the
following in a command window to build the code and executables:

C:>  nmake /f makefile.vc

Otherwise create your own VC++ project.  There aren't many files to deal with
here!


API 用法介绍

## 1. .shp file API
.shp api 使用一个SHPHandle代表一对打开的 .shp/.shx 文件对。SHPHandle的内容是可见的（参加shapefile.h），但应用程序应忽略。API函数旨在访问所有信息。
### 形状类型
形状具有与之对应的类型。以下是Shapefile支持的不同形状类型的列表。此时，shapefile中的所有形状都必须属于同一类型（NULL形状除外）。
#define SHPT_NULL 0
2D 形状类型（ArcView 3.x之前）：
  #define SHPT_POINT 1 点 
  #define SHPT_ARC 3 弧（多段线，可能在部分）
  #define SHPT_POLYGON 5 个多边形（可能是部分）
  #define SHPT_MULTIPOINT 8 多点（相关点）

  3D 形状类型（可能包括顶点的“测量”值）：

  #define SHPT_POINTZ 11
  #define SHPT_ARCZ 13
  #define SHPT_POLYGONZ 15
  #define SHPT_MULTIPOINTZ 18

  2D + 测量类型：

  #define SHPT_POINTM 21
  #define SHPT_ARCM 23
  #define SHPT_POLYGONM 25
  #define SHPT_MULTIPOINTM 28

  具有 Z 的复数（TIN 样）和测量：

  #define SHPT_MULTIPATCH 31
### SHP 对象
单个形状由 SHPObject 结构表示。使用 SHPCreateObject()、SHPCreateSimpleObject() 或 SHPReadObject() 创建的 SHPObject 应使用 SHPDestroyObject() 处理。
  类型定义结构
  {
    诠释 nSHPType; 形状类型（SHPT_* - 参见上面的列表）

    诠释 nShapeId; 形状编号（-1 未知/未分配）

    诠释 nParts；# of Parts（0 表示没有信息的单个部分）
    int *panPartStart; 零件的起始顶点
    int *panPartType; 零件类型（如果不是 SHPT_MULTIPATCH，则为 SHPP_RING）

    int nVertices; 顶点列表
    双 *padfX;
    双 *padfY;
    双 *padfZ; （如果未提供，则全部为零）
    双 *padfM; （如果未提供，则全部为零）

    双 dfXMin; X、Y、Z 和 M 维度的界限
    双 dfYMin;
    双 dfZMin；
    双 dfMMin；

    双 dfXMax；
    双 dfYMax；
    双dfZMax；
    双 dfMMax；
  } SHP 对象；
### SHPOpen()
SHPHandle SHPOpen( const char * pszShapeFile, const char * pszAccess );

  pszShapeFile：要访问的图层的名称。这可以是
			.shp 或 .shx 文件的名称，或者可以
			只是路径加上对的基本名称。

  pszAccess：fopen() 风格的访问字符串。此时只
			“rb”（只读二进制）和“rb+”（读/写二进制）
		        应该使用。
应该使用 SHPOpen() 函数来建立对用于访问顶点的两个文件（.shp 和 .shx）的访问。请注意，这两个文件都必须位于指定的目录中，并且必须具有小写的预期扩展名。返回的 SHPHandle 被传递给其他访问函数，并且应该调用 SHPClose() 来恢复资源，并在完成时将更改刷新到磁盘。
### SHPGetInfo()
void SHPGetInfo(SHPHandle hSHP, int * pnEntities, int * pnShapeType,
                 双 * padfMinBound, 双 * padfMaxBound );

  hSHP：之前由 SHPOpen() 返回的句柄
			或 SHPCreate()。

  pnEntities：指向整数的指针，其中的数量
			应放置实体/结构。可能为 NULL。

  pnShapetype：指向一个整数的指针，其中 shapetype
			应该放置这个文件的。形状文件可能包含
			SHPT_POINT、SHPT_ARC、SHPT_POLYGON 或
			SHPT_MULTIPOINT 实体。这可能是 NULL。

  padfMinBound：X、Y、Z 和 M 的最小值将被放入
                        这四个条目数组。这可能是 NULL。

  padfMaxBound：X、Y、Z 和 M 最大值将被放入
                        这四个条目数组。这可能是 NULL。
SHPGetInfo() 函数从整体上检索有关 shapefile 的各种信息。边界是从文件头读取的，如果文件生成不正确，可能会不准确。
### SHPReadObject()
SHPObject *SHPReadObject( SHPHandle hSHP, int iShape );

  hSHP：之前由 SHPOpen() 返回的句柄
			或 SHPCreate()。

  iShape：要读取的形状的实体编号。实体
			数字介于 0 和 nEntities-1 之间（如返回
			通过 SHPGetInfo())。
SHPReadObject() 调用用于从 shapefile 读取单个结构或实体。有关 SHPObject 字段的详细信息，请参阅 SHPObject 结构的定义。从 SHPReadObject() 返回的 SHPObject 应该使用 SHPDestroyShape() 释放。如果请求了非法的 iShape 值，SHPReadObject() 将返回 NULL。
请注意，放置在 SHPObject 中的边界是从文件中读取的边界，可能不正确。对于点，边界是从单个点生成的，因为通常不为点类型提供边界。

通常，返回的形状将是整个文件的类型。但是，任何文件也可能包含没有几何形状的 SHPT_NULL 类型的形状。一般来说，应用程序应该跳过而不是保留它们，因为它们通常表示交互式删除的形状。

### SHPClose()
无效 SHPClose(SHPHandle hSHP);

  hSHP：之前由 SHPOpen() 返回的句柄
			或 SHPCreate()。
SHPClose() 函数将关闭 .shp 和 .shx 文件，并将所有未完成的标头信息刷新到文件中。它还将恢复与句柄关联的资源。在此调用之后，hSHP 句柄不能再次使用。
SHPCreate()
SHPHandle SHPCreate( const char * pszShapeFile, int nShapeType );

  pszShapeFile：要访问的图层的名称。这可以是
			.shp 或 .shx 文件的名称，或者可以
			只是路径加上对的基本名称。

  nShapeType：新创建的要存储的形状的类型
			文件。它可以是 SHPT_POINT、SHPT_ARC、
		        SHPT_POLYGON 或 SHPT_MULTIPOINT。
SHPCreate() 函数将创建所需类型的新 .shp 和 .shx 文件。
### SHPCreateSimpleObject()
SHP 对象 *
     SHPCreateSimpleObject(int nSHPType, int nVertices,
			    双 *padfX, 双 * padfY, 双 *padfZ, );

  nSHPType：要创建的对象的 SHPT_ 类型，如
                        作为 SHPT_POINT 或 SHPT_POLYGON。

  nVertices：在 padfX 中传递的顶点数，
                        padfY 和 padfZ。

  padfX：nVertices 的数组，顶点的 X 坐标
                        对于这个对象。

  padfY：nVertices 的数组，顶点的 Y 坐标
                        对于这个对象。

  padfZ：nVertices 的数组，顶点的 Z 坐标
                        对于这个对象。这可能是 NULL 在这种情况下
		        它们都被假定为零。
SHPCreateSimpleObject() 允许方便地创建简单对象。这通常用于将 SHPObject 传递给 SHPWriteObject() 以将其写入文件。简单对象创建 API 假定每个顶点的 M（度量）值为零。对于复杂对象（例如多边形），假设只有一个部分，并且它是默认类型（SHPP_RING）。
对更复杂的对象使用 SHPCreateObject() 函数。SHPDestroyObject() 函数应该用于释放与使用 SHPCreateSimpleObject() 分配的对象相关联的资源。

此函数根据给定顶点计算 SHPObject 的边界框。

### SHPCreateObject()
SHP 对象 *
     SHPCreateObject（int nSHPType，int iShape，
                      int nParts，int * panPartStart，int * panPartType，
                      int nVertices，双 *padfX，双 * padfY，
                      双 *padfZ, 双 *padfM );

  nSHPType：要创建的对象的 SHPT_ 类型，如
                        作为 SHPT_POINT 或 SHPT_POLYGON。

  iShape：要使用此形状记录的 shapeid。

  nParts：此对象的零件数。如果这是
                        ARC 或 POLYGON 类型对象为零，单个
                        零值部分将在内部创建。

  panPartStart：环的从零开始的顶点列表
                        （部分）在这个对象。第一个应该永远是
                        零。如果 nParts 为 0，则这可能为 NULL。

  panPartType：每个部分的类型。这只有意义
                        对于 MULTIPATCH 文件。对于所有其他情况，这可能
                        为 NULL，并假定为 SHPP_RING。

  nVertices：在 padfX 中传递的顶点数，
                        padfY 和 padfZ。

  padfX：nVertices 的数组，顶点的 X 坐标
                        对于这个对象。

  padfY：nVertices 的数组，顶点的 Y 坐标
                        对于这个对象。

  padfZ：nVertices 的数组，顶点的 Z 坐标
                        对于这个对象。这可能是 NULL 在这种情况下
		        它们都被假定为零。

  padfM：nVertices M（测量值）的数组
			此对象的顶点。这可能是 NULL 其中
			在这种情况下，它们都被假定为零。
SHPCreateSimpleObject() 允许创建对象（形状）。这通常用于将 SHPObject 传递给 SHPWriteObject() 以将其写入文件。
SHPDestroyObject() 函数应该用于释放与使用 SHPCreateObject() 分配的对象关联的资源。

此函数根据给定顶点计算 SHPObject 的边界框。

### SHPComputeExtents()
void SHPComputeExtents(SHPObject * psObject);

  psObject：要就地更新的现有形状对象。
此函数将重新计算此形状的范围，根据形状的当前顶点集替换 dfXMin、dfYMin、dfZMin、dfMMin、dfXMax、dfYMax、dfZMax 和 dfMMax 值的现有值。此函数由 SHPCreateObject() 自动调用，但如果现有对象的顶点发生更改，则应再次调用它以修复范围。
### SHPWriteObject()
int SHPWriteObject( SHPHandle hSHP, int iShape, SHPObject *psObject );

  hSHP：之前由 SHPOpen("r+") 返回的句柄
			或 SHPCreate()。

  iShape：要写入的形状的实体编号。一个值
		        -1 应该用于新形状。

  psObject：要写入文件的形状。这个应该有
                        使用 SHPCreateObject() 创建，或
                        SHPCreateSimpleObject()。
SHPWriteObject() 调用用于将单个结构或实体写入 shapefile。有关 SHPObject 字段的详细信息，请参阅 SHPObject 结构的定义。返回值是写入形状的实体编号。
### SHPDestroyObject()
void SHPDestroyObject(SHPObject *psObject);

  psObject：要解除分配的对象。
当不再需要与 SHPObject 关联的资源时，应使用此函数解除分配，包括使用 SHPCreateSimpleObject()、SHPCreateObject() 创建并从 SHPReadObject() 返回的资源。
### SHPRewindObject()
int SHPRewindObject( SHPHandle hSHP, SHPObject *psObject );

  hSHP：shapefile（此时未使用）。
  psObject：要解除分配的对象。
此函数将反转任何必要的环，以便对 Shapefile 规范中所需的内环和外环的顺序强制执行 shapefile 限制。如果进行了更改，则返回 TRUE，如果未进行更改，则返回 FALSE。尽管可以传递任何对象，但只有多边形对象会受到影响。
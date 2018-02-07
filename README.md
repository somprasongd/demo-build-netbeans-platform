# การแก้ไข build script Netbeans Platform (Netbeans IDE 8.2) สำหรับการทำไฟล์ติดตั้ง

### การเพิ่ม RAM
1. เปิดไฟล์ project.properties โดยเลือกที่ชื่อโปรเจค > Important Files > Project Properties
2. เพิ่ม property `run.args.extra=-J-Xms32m -J-Xmx512m` ไว้ก่อน \"modules=\"
3. แก้ที build-launchers โดยเปิดไฟล์ suite.xml ถ้าเป็น Windows จะอยู่ที่ C:\Program Files\NetBeans 8.2\harness\suite.xml
4. เพิ่ม `<replace file="${app.conf}" token="-J-Xms24m -J-Xmx64m" value="${run.args.extra}" />` ระหว่าง

``` 
<property name="app.conf" location="${harness.dir}/etc/app.conf"/>
<replace file="${app.conf}" token="-J-Xms24m -J-Xmx64m" value="${run.args.extra}" />
<copy file="${app.conf}" tofile="${build.launcher.dir}/etc/${app.name}.conf" >
    <filterchain>
        <replacestring from="$${branding.token}" to="${branding.token}"/>
    </filterchain>
</copy>
```

### การเปลี่ยน Executable Icon บน Windows

ขั้นตอนปกติในการสร้าง app.exe นั้น Netbeans จะทำการ copy ไฟล์ C:\Program Files\NetBeans 8.2\harness\launchers\app.exe มาไว้ที่ [yourproject]\build\launcher\bin\ จากนั้นจะเปลี่ยนชื่อโดยใช้ชื่อจาก app.name ที่กำหนดไว้ใน project.properties แต่ไม่ทำการเปลี่ยน icon ให้

ดังนั้นเราต้องใช้ third-party utility program มาใช้ในการเปลี่ยน icon แทน เช่นใช้ [ReplaceVistaIcon.exe](http://www.rw-designer.com/compile-vista-icon) รันผ่าน command line `ReplaceVistaIcon.exe build\launcher\bin\<your branding name>.exe YourIconFile.ico`

**โดยมีวิธีการ ดังนี้**

1. เปิดไฟล์ C:\Program Files\NetBeans 8.2\harness\suite.xml
2. เพิ่มโค้ด `<project>...</project>` ดังนี้

```
<condition property="isWindows">
    <os family="windows" />
</condition>
<!-- Windows-only target that replaces the icon for the launcher exe with our own icon. -->
<target name="replaceWindowsLauncherIcon" if="isWindows" description="Replace the icon for the Windows launcher exe">
    <echo message="Replacing icon of Windows launcher executable."/>
    <exec executable="ReplaceVistaIcon.exe" resolveexecutable="true">
        <arg line="build/launcher/bin/${app.name}.exe ${app.name}.ico"/>
    </exec>
    <exec executable="ReplaceVistaIcon.exe" resolveexecutable="true">
        <arg line="build/launcher/bin/${app.name}64.exe ${app.name}.ico"/>
    </exec>
    <exec executable="ReplaceVistaIcon.exe" resolveexecutable="true">
        <arg line="build/launcher/bin/${app.name}_w.exe ${app.name}.ico"/>
    </exec>
</target>
```

3. แทรกโค้ด `<!-- Replace the icon for the Windows launcher exe. --><antcall target="replaceWindowsLauncherIcon"/>` หลังการ copy และ rename app.exe เสร็จแล้ว ดังนี้

``` 
<target name="build-launchers" depends="-init">
    <copy file="${harness.dir}/launchers/${app.exe.prefix}app.exe" tofile="${build.launcher.dir}/bin/${app.name}.exe" overwrite="true"/>
    <copy file="${harness.dir}/launchers/${app.exe.prefix}app64.exe" tofile="${build.launcher.dir}/bin/${app.name}64.exe" failonerror="false" overwrite="true"/>
    <copy file="${harness.dir}/launchers/${app.exe.prefix}app_w.exe" tofile="${build.launcher.dir}/bin/${app.name}_w.exe" failonerror="false" overwrite="true"/>
    <!-- Replace the icon for the Windows launcher exe. -->
    <antcall target="replaceWindowsLauncherIcon"/>
</target>
```
	

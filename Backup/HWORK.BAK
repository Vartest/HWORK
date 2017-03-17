(******************************* PDOX2WP.PAS *******************************)
program pdox2wp;
{$M 16384,0, 174080 }

(***************************************************************************)
uses pxengine, crt, dos;
(***************************************************************************)
const

   NetUserName ='H_INWORK';
   NetType     = OtherNet;
   (*   NetType=NotOnNet;   *)

   MaxClientFields= 14;

   HINWORK  = 'H_INWORK.DB';
   HEADERS  = 'HEADERS.DB';
   CLIENTS  = 'CLIENTS.DB';
   REPTEST  =  'REPTEST.WP';
   DummyFile=  'DUMMY.WP';

   MACYS    =  'MACYS';

   MAX_FIELDS = 13;


   TYPEOVER=FALSE;
   TYPEAFTER=TRUE;


           (* client table field lengths *)
   CLIENT_SIZE  = 8;
   FNAME_SIZE   = 15;
   LNAME_SIZE   = 20;
   ADDRESS_SIZE = 30;
   STATE_SIZE   = 2;
   ZIP_SIZE     = 10;
   PHONE_SIZE   = 12;

(***************************************************************************)
type

   mystrings = string[4];

   ctable= RECORD
      ClientName     : string[ CLIENT_SIZE ];
      FirstName      : string[ FNAME_SIZE  ];
      LastName       : string[ LNAME_SIZE  ];
      AddressLine1   : string[ ADDRESS_SIZE ];
      AddressLine2   : string[ ADDRESS_SIZE ];
      AddressLine3   : string[ ADDRESS_SIZE ];
      State          : string[ STATE_SIZE ];
      ZipCode        : string[ ZIP_SIZE ];
      PhoneNum       : string[ PHONE_SIZE ];
      FaxNum         : string[ PHONE_SIZE ];
   end;

   htable = RECORD
      TheClient     : string[ 6 ];
      TheAls        : string[ 8 ];
      Season        : string[ 20 ];
      Department    : string[ 32 ];
      Manufacturer  : string[ 32 ];
      Style         : string[ 46 ];
      FabricType    : string[ 32 ];
      Color         : string[ 32 ];
      TheComment    : string[ 255 ];
      Service       : string[ 20 ];
      Abbreviation  : string[ 255 ];
   end;



(***************************************************************************)
var
   ClientsTable : ctable;
   HeadersTable : htable;

   pxErr, sysErr, i: integer;

   wpSourceFile, wpDestFile: string;

   Tblhandle, c_tblhandle: tablehandle;
   Fldhandle, c_fldhandle: fieldhandle;
   Rechandle, c_rechandle: recordhandle;
   nflds: integer;
   nrecs: recordnumber;
   rk: char;
   exist, this_one: boolean;      { wp second file created variable }
   recs, max_recs: integer;
   Infile, outfile: file of char;
   f:text;

   NetDir,clientName, IniFileName, mesg: string;
   PdoxNetDir, TablesDir, SamplesDir, ClientsAtoLDir, ClientsMtoZDir: string;
   c_tablename, i_tableName: string;
              (* only used for MACYS *)
   styleString, POString, CenterString: string;
(***************************************************************************)
procedure capitalize( var buffer: string);
var
   i: byte;
begin
   for i:=1 to length(buffer) do buffer[i]:=upcase( buffer[i] );
end;
(************************************************************************)
function isDigit( ch: char):boolean;
var
   retVal:boolean;
begin
   if ch in ['0'..'9'] then retVal:=TRUE
   else retVal:=FALSE;
   isDigit:=retVal;
end;
(************************************************************************)
function isAlphaNumeric( ch: char):boolean;
var
   retVal:boolean;
begin
   if ch in [' ', '#', ','] then retVal:=FALSE
   else retVal:=TRUE;
   isAlphaNumeric:=retVal;
end;
(************************************************************************)
function copyString( theString: string; iPos, fPos:byte):string;
var
   i: byte;
   retVal: string;
begin
   retVal:='';
   for i:= iPos to fPos do retVal:= retVal + theString[i];
   copyString:= retVal;
end;
(************************************************************************)
procedure sysError( nameOFile: string);
var
   currentIniFile: string;
begin
   if (sysErr=163) then begin
      writeln('Sharing Violation: ERROR_NUM_163');
   end
   else begin
      writeln;
      writeln( 'Can not open file ', nameOFile, '.');
      writeln( 'Run-Time Error Number: ', sysErr ); clreol;
   end;
end;
(***************************************************************************)
function FindFile( fname: string; abyte:byte):boolean;
var
   sr : searchrec;
   i  : byte;
   retVal: boolean;
begin
   retVal:=FALSE;

   for i:=1 to length( fname) do fname[i]:=upcase( fname[i]);
   findfirst( fname, abyte, sr);
   if doserror=0 then retVal:= TRUE
   else if doserror=18 then retVal:= FALSE
   else if doserror=3 then
   begin
      writeln('Could not find ', fname,'.');
      retVal:= FALSE;
   end;

   FindFile:= retVal;
end;
(*************************************************************************)
procedure printf( outstring: string);
begin
   writeln(    outstring);
   writeln( f, outstring);
end;
(***************************************************************************)
procedure cputs( outstring: string);
begin
   write(    outstring);
   write( f, outstring);
end;
(***************************************************************************)

procedure EngError( pxErr:integer);
begin
   writeln(f);
   writeln(f, PxErrMsg( pxErr));
   flush( f);
end;
(***************************************************************************)
function removeBlanks( astring: string): string;
var i: byte;
begin
   while ( pos( ' ',astring) <>0 ) do begin
        i:=pos( ' ', astring );
        delete( astring, i, 1 );
   end;
   removeBlanks:= astring;
end;
(***********************************************************************)
procedure readpaths;
const
   IniFileName= 'HWORK.INI';
var
   fhandle : text;
begin
   if ( FindFile( IniFileName, 0) ) then begin
      assign( fhandle, IniFileName);    {$i-}
      reset( fhandle);
      sysErr:=ioresult;                 {$i+}
      if (sysErr<>0) then sysError( IniFileName );
           (* global strings are assigned the corresponding paths *)
      readln( fhandle, tablesDir);
      readln( fhandle, samplesDir);
      readln( fhandle, ClientsAtoLDir);
      readln( fhandle, ClientsMtoZDir);
      readln( fhandle, PdoxNetDir);

      close ( fhandle);
             (* remove Any blank spaces *)
      TablesDir := removeBlanks( TablesDir );
      SamplesDir:= removeBlanks( samplesDir );
      ClientsAtoLDir := removeBlanks( ClientsAtoLDir );
      ClientsMtoZDir := removeBlanks( ClientsMtoZDir );
   end;
end;
(**************************************************************************)
function VerifyAlsNum( SampleAls: String):Boolean;
var
   RetVal, continue: Boolean;
   i : byte;
   theChar: char;
begin
   RetVal:=TRUE;
   continue:=TRUE;

   if ( Length( SampleAls) >=7 ) then begin
      i:=1;
      while (continue) do begin
         theChar:= SampleAls[ i ];

         if (theChar in [ '0'..'9']) then inc( i)
         else begin
            RetVal:=FALSE;
            continue:=FALSE;
         end;
         if (i>=7) then continue:=FALSE;
      end;

      if (SampleAls[ 7 ] in ['0', '9']) then RetVal:=FALSE;

   end
   else begin
      printf('   *** Note: Length of als is less than 7 characters.');
      RetVal:=FALSE;
   end;

   VerifyAlsNum:= RetVal;
end;
(**************************************************************************)
procedure FileManage( var fullfilename: string; var exist:boolean);
var
   sr:searchrec;
   i:integer;
   ch:char;
   fullname   :string[8];
   drive :string;
   phy_drive:byte;
begin

   exist:=TRUE;                       (* assume: file already exists *)


   fullname:= HeadersTable.TheAls;    (* copy als field *)

   fullfilename :=  fullname + '.wp'; (* add extesion .wp to fullname *)

            (* copy client name *)
   clientName:= HeadersTable.TheClient;
   ch:= clientName[1];

   if ch in ['A'..'L'] then begin
      drive:= ClientsAtoLDir;
   end
   else begin
      drive:= ClientsMtoZDir;
   end;

   fullfilename:= drive + '\' + clientName + '\' + fullfilename;

   for i:=1 to length(fullfilename) do
               fullfilename[i]:=upcase(fullfilename[i]);

   findfirst( fullfilename, 0, sr);
   if (doserror=0) then begin
      drive:= 'File '  +  fullfilename +  ' already exists.';
      printf( drive);
   end
   else if (doserror=18) then begin
      drive:= 'Created ' + fullfilename + '     ';
      cputs( drive);
      exist:=FALSE;
   end
   else if (doserror=3)  Then begin
      drive:= '   Could not create '   + fullfilename + '.  Check Path!';
      printf( drive );
   end;


   if ( NOT VerifyAlsNum( HeadersTable.TheAls)) then begin
      printf('   Encountered non-standard ALS format:');
      printf('                ' + HeadersTable.TheClient + '  '
                                + HeadersTable.TheAls);
      printf('   ');
   end;

end;
(****************************************************************************)
(* shifts a string of n chars left once and adds a char in the nth position *)

procedure ShiftStrLeft(var a: string;  n:integer; ch:char);
var
   i :integer;
   b : string;
begin
  For i:=2 to n do b[i-1]:=a[i];
  B[n]:=ch;
  for i:=1 to n do a[i]:=b[i];
end;
(***************************************************************************)
procedure placeString( sourcestr:string);
var i:integer;
begin
  for i:=1 to length(sourcestr) do  write( outfile, sourcestr[i]);
  write('.');
end;
(***************************************************************************)
(* * Replaces TargetString with NewString in wp file ;
   * n=length of the targetstr looking for in the wp source file
   *************************************************************************)
procedure FindWord(  TargetString: string; NewString:string);
var
   ch     : char;
   same   : boolean;
   CompString: string;
   n      : byte;
   cur_in_pos, cur_out_pos : longint;

begin
   same:=FALSE;
   n:= length( TargetString);
   for i:=1 to 21 do CompString[i]:=' ';   (* CompString is filled  *)
   CompString[0]:= chr( n);             (* same length as targetstring *)

   cur_in_pos:=filepos( infile);
   cur_out_pos:=filepos( outfile);

   While ( (NOT EOF(infile)) AND (NOT same) ) do  begin
      read( infile, ch);
      write( outfile, ch);

      ShiftStrLeft( CompString, n, ch);

      same:= TargetString=CompString;

      if (same) then begin
         seek( outfile, filepos(outfile) - n );
         placeString( NewString);
      end;
   end;

   if (NOT same) then
   begin
                      (* place file ptr back to what it was originally *)
      seek( infile, cur_in_pos);
      seek( outfile, cur_out_pos);


      printf('   ');
      CompString:= 'Did Not find±'+ TargetString + '±';
      printf( CompString);

   end;

end;
(***************************************************************************)
procedure CopyRestOfDocument;
var
   ch: char;
begin
   While (NOT EOF(infile)) do  begin
      read( infile, ch);
      write( outfile, ch);
   end;
   printf(' ');
end;
(**************************************************************************)

(*  * Opens up a wordperfect file if a the fileName doesn't already Exist
    * Returns False if File doesn't Exist
    ***********************************************************************)
procedure OpenWpFiles( var exist:boolean);
var
   sysErr: integer;
   ClientSampleFile: String;

begin

   filemanage( wpDestFile, exist);     {* do file management.  *}
          (* If File already exists then exit procedure *)
   if (Exist=TRUE) then Exit;

   ClientSampleFile:= SamplesDir + '\' + ClientName + '.WP';
            (* Does this client have her own Sample Reptest File *)
   if ( NOT FindFile( ClientSampleFile, 0 ) ) then begin
            (* Use default Reptest File *)
      ClientSampleFile:= SamplesDir + '\' + REPTEST;

      if ( NOT FindFile( ClientSampleFile, 0 ) ) then begin
         mesg:= 'Could NOT find sample file: ' + ClientSampleFile;
         printf( mesg);
         exit;
      end;
   end;

      (** open WP File For Input **)
   wpSourceFile:= ClientSampleFile;
   assign( infile, wpSourceFile);     {$i-}
   reset( infile);                    {$i+}

   sysErr:=ioresult;
   if (sysErr<>0) then begin
      printf('   Can NOT open wpSourceFile');
      halt;
   end;

      (** Open Wp File for output **)
   assign( outfile, DummyFile);      {$i-}
   rewrite( outfile);                 {$i+}
   sysErr:=ioresult;
   if (sysErr<>0) then begin
      printf('   Can NOT open wpDestFile');
      halt;
   end;
end;
(***************************************************************************)
function GetAlphaField( var RHandle : RECORDHANDLE;
                            FHandle : FIELDHANDLE): string;
var
   avalue: string;
   fldhandle: FieldHandle;
begin
             (* returns string from an alphanumeric field *)
   PxErr:= PxGetAlpha( RHandle, FHandle, avalue);
   if (pxErr<>PXSUCCESS) then engError( pxErr);

   GetAlphaField:= avalue;
end;
(***************************************************************************)
Procedure HServicePriority;
var
   sp: string;
begin
   sp:= HeadersTable.Service;
   for i:=1 to length( sp ) do sp[i]:=upcase( HeadersTable.Service[i] );
   i:=pos('RED L', sp);
   if (i>0) then begin
      sp:= 'RED LABEL SERVICE';
      FindWord( 'ServiceP', sp )
   end
   else begin
      i:=pos('BLUE L', sp);
      if (i>0) then begin
         sp:= 'BLUE LABEL SERVICE';
         FindWord( 'ServiceP', sp )
      end
      else FindWord( 'ServiceP', sp );
   end;
end;
(***************************************************************************)
procedure XferClientsRecord;
var
   target: string;
   StringSize: byte;
begin

   With ClientsTable do begin
      ClientName := GetAlphaField( c_rechandle, 1);
      FirstName  := GetAlphaField( c_rechandle, 2);
      LastName   := GetAlphaField( c_rechandle, 3);

      AddressLine1:= GetAlphaField( c_rechandle, 6);
      AddressLine2:= GetAlphaField( c_rechandle, 7);
      AddressLine3:= GetAlphaField( c_rechandle, 8);

      state   := GetAlphaField( c_rechandle, 9);
      ZipCode := GetAlphaField( c_rechandle, 10);


      target:='ADDL1';

      StringSize:= length( AddressLine1) - length(target);
      For i:=1 to StringSize do target:= target + ' ';

      FindWord( Target,   AddressLine1 );
      FindWord(  'MCFN',  FirstName );
      FindWord(  'MCLN',  LastName );
      FindWord(  'ADDL2', AddressLine2 );
      FindWord(  'ADDL3', AddressLine3 );
      FindWord(  'STATE', State );
      FindWord(  'ZIP',   ZipCode );

   end;
end;
(***************************************************************************)
(* * returns the paramaters of the first numeric string encountered
     in the buffer
   * start_index is the pos to begin search from
   *************************************************************************)
procedure RetNumericFieldParams( theBuffer: string; start_index: byte;
                                 var iPos, fPos: byte);
var
   i, bufferLength: byte;
   continue, inDigitString: boolean;
   ch: char;
begin
   i:= start_index;
   bufferLength:= length( theBuffer);

   inDigitString:=FALSE;
   continue:=TRUE;
   while ( continue) do begin

      ch:= theBuffer[i];

      if (isAlphaNumeric( ch)) then begin
         if (NOT inDigitString) then begin
            inDigitString:=TRUE;
            iPos:= i;
         end;

      end
      else begin
         if (inDigitString) then begin
            fPos:= i-1;
            continue:=FALSE;
         end;
      end;

      if (continue) then inc( i);
      if (i>bufferLength) then continue:=FALSE;
   end;

   if (inDigitString) then begin
      fPos:= i-1;
   end;

end;
(***************************************************************************)
procedure GetStartAndEndPos( SubString, SourceString: string;
                             var iPos, fPos: byte);
var
   i, start_at: byte;
begin
   iPos:=0;
   fPos:=0;

   i:= pos( subString, SourceString);
   if (i>0) then begin
      start_at:= i + length( SubString);
      RetNumericFieldParams( SourceString, start_at, iPos, fPos);
   end;
end;
(***************************************************************************)
(*  gets   'PO #', 'CENTER #', and 'STYLE #'   *)
procedure ParseForFields;
var
   StyleField, tempString: string;
   iPos, fPos: byte;
begin
   POString:='';
   styleString:='';
   CenterString:='';

   StyleField:= HeadersTable.Style;

   tempString:= StyleField;
   capitalize( tempString);

   GetStartAndEndPos( 'STYLE', tempString, iPos, fPos);
   if (iPos<>fPos) then styleString:= copyString( tempString, iPos, fPos);

   GetStartAndEndPos( 'PO', tempString, iPos, fPos);
   if (iPos<>fPos) then POString:= copyString( tempString, iPos, fPos)
   else begin
      GetStartAndEndPos( 'P.O.', tempString, iPos, fPos);
      if (iPos<>fPos) then POString:= copyString( tempString, iPos, fPos)
   end;

   GetStartAndEndPos( 'CENTER', tempString, iPos, fPos);
   if (iPos<>fPos) then CenterString:= copyString( tempString, iPos, fPos);

end;
(***************************************************************************)
procedure XferMacysRecord;
var
   target: string;
   StringSize: byte;
begin

   With ClientsTable do begin
      ClientName := GetAlphaField( c_rechandle, 1);
      FirstName  := GetAlphaField( c_rechandle, 2);
      LastName   := GetAlphaField( c_rechandle, 3);

      AddressLine1:= GetAlphaField( c_rechandle, 6);
      AddressLine2:= GetAlphaField( c_rechandle, 7);
      AddressLine3:= GetAlphaField( c_rechandle, 8);

      state   := GetAlphaField( c_rechandle, 9);
      ZipCode := GetAlphaField( c_rechandle, 10);


      ParseForFields;

      target:= POString;
      FindWord(  'PO_#', target );

      target:= CenterString;
      FindWord(  'C_NUM', target);

              (* first address line is special. Replace equal characters *)
      target:='ADDL1';
      StringSize:= length( AddressLine1) - length( target );
      For i:=1 to StringSize do target:= target + ' ';

      FindWord( target, AddressLine1 );

      FindWord( 'MCFN',  FirstName );
      FindWord( 'MCLN',  LastName );
      FindWord( 'ADDL2', AddressLine2 );
      FindWord( 'ADDL3', AddressLine3 );
      FindWord( 'STATE', State );
      FindWord( 'ZIP',   ZipCode );
      FindWord( 'DpMt',  HeadersTable.Department  );
   end;
end;
(***************************************************************************)
(* * differentiates between clients and obtains info for that client.
   * searches CLIENTS.DB for the right client
   *************************************************************************)
procedure WhichClient;
var
   alpha :string;
   looking4: string;
   look4, same  : boolean;
begin

          (* opens CLIENTS.DB  table *)
   Pxerr:=pxtblopen( c_tablename, c_tblhandle, 0, FALSE);
   if (pxErr<>PXSUCCESS) then engError( pxErr);


   Px( pxrecbufopen(c_tblhandle, c_rechandle));

   look4:=FALSE;

   repeat
           (* place current record's client field into looking4 var *)
      Looking4 := HeadersTable.TheClient;

      Px( PXFldHandle( c_tblHandle, 'CLIENTAB', C_fldhandle));

      PX( PxPutAlpha( c_recHandle, C_Fldhandle, LOOKING4));
      PxErr:= PXSrchFld (C_TblHandle, C_RecHandle, C_FldHandle, SearchFirst);
      if (pxErr<>PXSUCCESS) then
         If (PxErr=PXERR_RECNOTFOUND) then begin

            printf('   ');
            mesg:= '   *** Note: Did not find ' + looking4 +  ' in CLIENTS.DB.';
            printf(mesg);
            printf('   ');

            look4:=TRUE;  (* set TRUE to exit REPEAT loop *)
         end
      else engError( pxErr);


      PX( PxRecBufEmpty(C_RecHandle));

            (* xfer current rec into rec xfer buffer *)
      PXERR:=PxRecGet(C_TblHandle, C_RecHANDLE);
      if (pxErr<>PXSUCCESS) then engError( pxErr);

      c_fldHandle:=1;
      pxErr:=PXGetAlpha( c_RecHandle, c_FldHandle, alpha);
      if (pxErr<>PXSUCCESS) then engError( pxErr);

      if (alpha=looking4) then begin

         if (looking4=MACYS) then begin
            XferMacysRecord;
         end
         else  XferClientsRecord;

         look4:=TRUE;
      end;
   until look4;

   px( pxrecbufclose(c_rechandle));
              (* close the clients table *)
   Px(pxtblclose( c_tblhandle));

end;
(***************************************************************************)
procedure TransferToWp;
var
   n:integer;
   target: string;
begin
   with HeadersTable do begin

      gotoxy( 60, wherey);
      FindWord( 'CLIENT',  TheClient);
      FindWord( 'ALS', TheAls );

          (* copy Client Information *)
      WhichClient;

      if (ClientName=MACYS) then begin
         FindWord( 'MANUFACT',   Manufacturer );

         target:= StyleString;
         FindWord( 'STYLE#',     target);

         FindWord( 'FABRICTYPE', FabricType );
         FindWord( 'COLOUR',     Color );
         FindWord( 'SEASON',     Season );
         FindWord( 'S/COMMENT',  TheComment );
         FindWord( 'ABBRS:',     Abbreviation );
         HServicePriority;
      end
      else begin
         FindWord( 'SEASON',     season );
         FindWord( 'DEPARTMENT', Department );
         FindWord( 'STYLE',      style );
         FindWord( 'MANUFACT',   manufacturer );
         FindWord( 'FABRICTYPE', FabricType );
         FindWord( 'Color',      Color );
         FindWord( 'Submit/',    TheComment );
         FindWord( 'ABBRS:',     Abbreviation );
         HServicePriority;
      end;

      CopyRestOfDocument;
   end;
end;
(***************************************************************************)
(* reads an alpha field from the current record *)
procedure ReadField( var rechandle: recordhandle; var fldhandle:fieldhandle);
var
   avalue:string;
begin
       (* returns string from an alphanumeric field *)
   Pxerr:=pxgetalpha(rechandle, fldhandle, avalue);
   if (pxErr<>PXSUCCESS) then engError( pxErr);

        (* place string into my record array *)
end;
(***************************************************************************)
(* reads an entrie record from table where the cursor is pointing *)
procedure ReadRecord;
begin
   with HeadersTable do begin
      TheClient    := GetAlphaField( rechandle, 1);
      TheAls       := GetAlphaField( rechandle, 2);
      season       := GetAlphaField( rechandle, 4);
      department   := GetAlphaField( rechandle, 5);
      manufacturer := GetAlphaField( rechandle, 6);
      style        := GetAlphaField( rechandle, 7);
      FabricType   := GetAlphaField( rechandle, 8);
      color        := GetAlphaField( rechandle, 9);
      TheComment   := GetAlphaField( rechandle, 11);
      service      := GetAlphaField( rechandle, 12);
      abbreviation := GetAlphaField( rechandle, 13);
   end;
end;
(***************************************************************************)
function DOSCopy( sString, dString: string):boolean;
var
    cmdline: string;
    i: integer;
    x,y:byte;
begin
   DOSCopy:=FALSE;

   i:=1;
   WHILE ( i>0 ) DO BEGIN
       i:=pos(' ', sString);
       if (i>0) then delete( sString, i, 1 );
   END;

   i:=1;
   WHILE ( i>0 ) DO BEGIN
       i:=pos(' ', dString);
       if (i>0) then delete( dString, i, 1 );
   END;


           (* Exec command messes up border rectangle        *)
           (* therefore redirect its output to file junk.dat *)
   cmdline:='/C copy ' + sString + ' ' + dString +  ' > NUL';
   SwapVectors;
   Exec( getEnv('COMSPEC'), cmdline );
   SwapVectors;

   if (DosError<>0) then begin
         printf( cmdline);
         str( dosError, cmdline);
         printf('DOS copy returned run-time error #: ' + cmdline);
   end
   else begin
         Writeln( cmdline );
         DOSCopy:=TRUE;
   end;
end;
(**************************************************************************)
procedure RenameTheFile;
var
   copied: boolean;
begin
   copied:=DosCopy( 'DUMMY.WP',  wpDestFile);
end;
(***************************************************************************)
procedure closefiles;
begin
   Close( infile);
   Close( outfile);
   RenameTheFile;
end;
(***************************************************************************)
procedure checkTable( var theTable: string);
begin
   if ( FindFile( theTable, 0) )  then writeln( 'Found table: ', theTable)
   else begin
      clrscr;
      gotoxy( 1, 5);
      writeln('Unable to locate table ', theTable);
      writeln('Hint: ');
      writeln('   Ensure the above path is correct.');
      writeln('   Ensure that the first line in  the file PDOX2WP.INI holds ');
      writeln('        the correct path to the tables CLIENTS, and H_INWORK.');
      writeln;
      writeln('Can not proceed. Exiting out!');
      halt;
   end;
end;
(***************************************************************************)
procedure checkPath( var thePath: string);
begin
   if ( FindFile( thePath, $10) ) then writeln( 'Found Directory: ', thePath)
   else begin
      clrscr;
      gotoxy( 1, 5);
      writeln('Unable to locate directory ', thePath);
      writeln('Hint :');
      writeln('   Ensure that the PDOX2WP.INI file holds the path of the correct directory');
      writeln;
      writeln('Can not proceed. Exiting out!');
      halt;
      end;
end;
(***************************************************************************)
procedure VerifyDirectories;
begin
   i_tableName:= TablesDir + '\' + HINWORK;
   c_tableName:= TablesDir + '\' + CLIENTS;


   capitalize( i_tableName);
   capitalize( c_tableName);
   capitalize( SamplesDir);
   capitalize( ClientsAtoLDir);
   capitalize( ClientsMtoZDir);

   checkTable( i_TableName);
   checktable( c_TableName);

   checkPath( ClientsAtolDir);
   checkPath( ClientsMtoZDir);
   checkpath( SamplesDir);

end;
(***************************************************************************)
   (* strip Network Dir from the PARADOX.NET string *)
function VerifyNetDir:boolean;
var
   i, j : byte;
   RetVal : boolean;
begin
   RetVal:= FALSE;
   j:= length(PdoxNetDir);
   for i:=1 to j do PdoxNetDir[i]:=upcase( PdoxNetDir[i] );

          (* Does PdoxNetDir string hold filename and extension? *)
   i:= pos('PARADOX.NET', PdoxNetDir);
   if (i=0) then begin
      if PdoxNetDir[ j ]= '\' then PdoxNetDir:= PdoxNetDir + 'PARADOX.NET'
      else PdoxNetDir:= PdoxNetDir + '\PARADOX.NET';
   end;

          (* Verify existance of file PARADOX.NET *)
   if ( FindFile( PdoxNetDir, 0) ) then begin
      NetDir:= PdoxNetDir;

      j:= pos('PARADOX.NET', NetDir);
      NetDir[0] := chr( j-1 );

      RetVal:=TRUE;
   end;

   VerifyNetDir:= RetVal;
end;
(**************************************************************************)

(**************************************************************************)
    (* returns false if failed to initialize engine on a net *)
procedure init_process( var ok: boolean);
begin
   ok:=FALSE;

   if ( VerifyNetDir ) then begin
                   (* initialize engine on network  *)
      pxErr:= PXNetInit( NetDir, NetType, NetUserName);
      if (pxErr=PxSuccess) then begin
                     (* writeln(f, 'Initialized Engine on network'); *)
         ok:=TRUE;
      end
      else begin
         if (pxErr=PXERR_ALREADYINIT) then engError( pxErr)
         else begin
            engError( pxErr);
            writeln(f, 'Unable to initialize Engine Environment!');
            writeln(f, 'Killing the running process.');
         end;
      end;
   end
   else begin
      writeln(f, 'Net Directory is not a valid Directory.');
      Writeln(f, 'Don''t know path to PARADOX.NET. Exiting Out.');
   end;
end;
(**************************************************************************)
procedure pdoxEngine;
var
   continue: boolean;

begin

   VerifyDirectories;

                         (* px(pxinit);   (* initialize engine *)
   init_process( continue );
   if (continue=FALSE) then begin
      writeln('Procedure init_process returned FALSE. Exiting.');
      halt;
   end;

                (* opens table h_inwork *)
   px( pxtblopen( i_tableName, tblhandle, 0, FALSE) );

                 (* returns # of fields in table *)
   pxErr:= pxrecnflds( tblhandle, nflds); 
   if (pxErr<>PXSUCCESS) then engError( pxErr)
   else writeln( 'Number of fields in table ',
                 i_tablename, ' is: ', nflds);

                 (* return number of recs in table *)
   PxErr:= pxtblnrecs( tblhandle, nrecs);
   if (pxErr<>PXSUCCESS) then engError( pxErr)
   else begin
      str( nrecs, mesg);
      mesg:= 'Number of records: ' + mesg;
      printf( mesg);
   end;

                 (* create  rec transfer buffer  *)
   px( pxrecbufopen(tblhandle, rechandle));
                 (* return field handle *)
   px( pxfldhandle(tblhandle, 'client', fldhandle) );

   max_recs:=nrecs;

   recs:=0;
   repeat
                  (* transfer current record to the record buffer *)
      Px( pxrecget(tblhandle, rechandle));

                  (* read fields into a record *)
      ReadRecord;


                   (* open wp files. Returns FALSE if opened up wp files *)
      OpenWpFiles( exist);

                   (* transfer the record to the wp file *)
      if (NOT exist) then TransferToWp;

                   (* clears the record transfer buffer *)
      pxerr:= pxrecbufempty( rechandle);
      if (pxErr<>PXSUCCESS) then engError( pxErr);
                   (* move to the next record of a table *)
      pxerr:= pxrecnext(tblhandle);
      if (pxErr<>PXSUCCESS) then engError( pxErr);

      recs:=recs+1;
      if (not exist) then closefiles;   (* close wp files *)

   until (recs>=max_recs);

                    (* removes a record transfer buffer *)
   px( pxrecbufclose(rechandle));

   Px( pxtblclose( tblhandle));    (* close the answers table *)

                     (* close engine environment *)
   Px(pxexit);                    

end;
(********************** main program ****************************************)
begin
   clrscr;
   assign(f, 'DATA.DAT');       {$i-}
   rewrite(f);                  {$i+}

   ReadPaths;


   if (ioresult<>0) then begin
      writeln('Can''t open file DATA.DAT for output.');
      writeln('Error number: ', ioresult );
   end;

   pdoxEngine;


   writeln(f, 'Every file closed properly');
   close(f);                      (* close output file *)
end.
(***************************************************************************)



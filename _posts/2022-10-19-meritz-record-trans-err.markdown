---
title:  "[delphi] 메리츠화재 녹취 전송 오류"
layout: single
categories: 
    - delphi
comments: true
date: 2022-10-18
last_modified_at: 2022-10-19
author: "Lee Cool"
permalink: categories/delphi/recordTrans
---
  

#  [이전 작업]
##  1. 기존 프로세스
> 1) beforeWork 에서 FTP 연결
> 2) GetFileWork 에서 녹취 복호화 후 코덱 변환하여 전송

## 2. 발생 이슈
2022-10-17 15:43 이후부터 녹취 전송 실패
> __[프로그램 로그]__  
[2022-10-18 09:09:49.152]->복호화 Err -> Socket Error # 10054  
Connection reset by peer.

구글링 해본 결과 해당 오류의 Exception 이유를 찾았다.
- 원격 서버에서 Connection을 reset 처리
- 종료된 커넥션을 재사용하려고 할 때
- 클라이언트에서 정지버튼을 누르거나, 브리아주를 종료하거나, 다른 화면으로 이동 등의 이유로 서버측에서 작업 결과를 전달할 곳이 없어졌을 때
- Connection 에서 TimeOut 발생
- 메모리 부족
- 소켓 고갈 등등
##### 출처 : https://devlog.changhee.me/posts/2일간_씨름했던__Connection_reset_by_peer/
<br>
이 중에서 녹취가 큰 파일인 경우에만 오류가 발생하기 때문에 나의 상황에서는 Connection TimeOut 이 의심된다.  
<br>

# [작업 내용]
## 1. BeforeWork 에서 Connect 주석처리
> - FTP 정보 셋팅 후, 연결 부분은 제외

```delphi
try
    //FRecordFTP.Connect;    주석처리

    if gsFTPCharSet='UTF8' then
      FRecordFTP.IOHandler.DefStringEncoding := TEncoding.UTF8
    else
      FRecordFTP.IOHandler.DefStringEncoding := TEncoding.GetEncoding(949);

  except
    on e : Exception do
    begin
      if FRecordFTP.Connected then
        FRecordFTP.Disconnect;

      prglog('RecordFTP error : ' + e.Message);
    end;
  end;
```

## 2. GetFileWork 에서 전송 직전에 FTP 연결
> 녹취 전송 Put을 하기 직전에 connect  
> 복호화 작업에서 시간이 오래걸릴 경우 전송할 때 FTP 연결이 끊어지는 경우를 막고자 녹취 전송 직전 FTP 연결하도록 수정


```delphi
function TMeritz.GetFileWork(Source, Dest: string): boolean;
const
  _IF_HANIL_RTS_04 = 24;

var
  sSendMsg : string;
  RS : Boolean;
  ss, MS: TMemoryStream;
begin
  Result := False;

  RS := False;

  FSEED := TRecEncrypt.Create(ExtractFilePath(ParamStr(0)) + 'SEEDKEY.key');
  try
    sSendMsg := '';

    prglog(Format('Job File : %s -> %s', [Source, Dest]));
    if Assigned(FMehtod) then
    begin
      gSFile := Source;
      gDFile := Dest;
      Synchronize(FMehtod);
    end;

    if not FileExists(Source) then
    begin
      prglog(Format('Source file not exists...filename : %s', [Source]));
      exit;
    end;

    //Record File FTP Put
    if Dest <> '' then
    begin
      if Assigned(FRecordFTP) then
      begin
        //if FRecordFTP.Connected then
        begin
          try
            prglog(Format('gsRecordEncrypt : %s ', [gsRecordEncrypt]));
            if gsRecordEncrypt = 'Y' then
            begin
              MS := TMemoryStream.Create;
              try
                try
                  if FSEED.DecryptRecfile(Source, MS) then
                  begin
                    if UpperCase(ExtractFileExt(Source)) = _ExtNTS then
                    begin
                      ss := NTSFilecheck(MS);
                      try
                        //if FRecordFTP.Connected then
                        begin
                          try
                            prglog('FTP 연결 되어있음');
                            FRecordFTP.Connect;

                            if gsFTPCharSet='UTF8' then
                              FRecordFTP.IOHandler.DefStringEncoding := TEncoding.UTF8
                            else
                              FRecordFTP.IOHandler.DefStringEncoding := TEncoding.GetEncoding(949);
                            if FRecordFTP.Connected then
                            begin
                              FRecordFTP.Put(ss, Dest, False);
                            end;
                            FRecordFTP.Disconnect;
                          except
                            on e : Exception do
                              prglog('RecordFTP put error : ' + e.Message);
                          end;
//                        end else begin
//                          prglog('FTP 연결 실패!');
                        end;
                      finally
                        ss.Free;
                      end;
                    end
                    else
                    begin
                      try
                       // if FRecordFTP.Connected then
                        begin
                          try
                            prglog('FTP 연결 되어있음');
                            FRecordFTP.Connect;

                            if gsFTPCharSet='UTF8' then
                              FRecordFTP.IOHandler.DefStringEncoding := TEncoding.UTF8
                            else
                              FRecordFTP.IOHandler.DefStringEncoding := TEncoding.GetEncoding(949);

                            if FRecordFTP.Connected then
                            begin
                              FRecordFTP.Put(ss, Dest, False);
                            end;
                            FRecordFTP.Disconnect;
                          except
                            on e : Exception do
                              prglog('RecordFTP put error : ' + e.Message);
                          end;
//                        end else begin
//                          prglog('FTP 연결 실패!');
                        end;
                      finally
                        MS.Free;
                      end;
                    end;
                    RS := True;
                    prglog(Format('%s , %s', [FRecordFTP.LastCmdResult.Code, FRecordFTP.LastCmdResult.Text.Text]));
                    prglog(Format('복호화 전송 완료 -> Dest : %s ', [Source]));
                  end
                  else
                    prglog(FSEED.ErrMsg);
                except
                  on e : Exception do
                  begin
                    prglog(Format('복호화 Err -> %s ', [e.message]));
                  end;
                end;
              finally
                MS.Free;
              end;
            end
            else
            begin
              if UpperCase(ExtractFileExt(Source)) = _ExtNTS then
              begin
                ss := NTSFilecheck(MS);
                try
                  FRecordFTP.Put(ss, Dest, False)
                finally
                  ss.Free;
                end;
              end
              else
                //녹취파일
                FRecordFTP.Put(Source, Dest, False);

              RS := True;
              prglog(Format('그냥 전송 완료 -> Source : %s ', [Source]));
            end;
          except
            on e : Exception do
            begin
              prglog('Record Put Error ->'+e.Message);
            end;
          end;
        end;
      end;
    end;

    //Record Index FTP Put
    if (FIdxString <> '') and (RS) then
    begin
      try
        sSendMsg := GetBitsValue(FIdxString, _IF_HANIL_RTS_04);
        with TBitsIF.Create do
        begin
          try
            if IdHttpPost(gsSendUrl, sSendMsg) then
            begin
              Result := True;
              prglog('RecIdx Send');
            end;
          finally
            Free;
          end;
        end;
      except
        on e : Exception do
        begin
          prglog('RecIdx Put Error ->'+e.Message);
        end;
      end;
    end;

    if not Result then prglog(Format('FTP Put File Fail : %s ', [Source]));
  finally
    FSEED.Free;
  end;
end;
```
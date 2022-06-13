![Maintained YES](https://img.shields.io/badge/Maintained%3F-yes-green.svg?style=flat-square&color=important)
![Memory Leak Verified YES](https://img.shields.io/badge/Memory%20Leak%20Verified%3F-yes-green.svg?style=flat-square&color=important)
![Stars](https://img.shields.io/github/stars/antoniojmsjr/FastReportExport.svg?style=flat-square)
![Forks](https://img.shields.io/github/forks/antoniojmsjr/FastReportExport.svg?style=flat-square)
![Issues](https://img.shields.io/github/issues/antoniojmsjr/FastReportExport.svg?style=flat-square&color=blue)
![Release](https://img.shields.io/github/v/release/antoniojmsjr/FastReportExport?label=Latest%20release&style=flat-square&color=important)

# FastReportExport

**FastReportExport** é uma biblioteca para exportação de relatórios com [Fast Report](https://www.fast-report.com) para ambientes **multithreading**.

Implementado na linguagem Delphi, utiliza o conceito de [fluent interface](https://en.wikipedia.org/wiki/Fluent_interface) para guiar no uso da biblioteca, desenvolvido para exportar relatórios nos formatos PDF, HTML, PNG, entre outros, conforme a necessidade.

**Ambientes**

* Windows Forms
* Windows Console
* Windows Service
* IIS ISAPI[(Horse)](https://github.com/HashLoad/horse)
* IIS CGI[(Horse)](https://github.com/HashLoad/horse)

## ⭕ Pré-requisito

Para utilizar o **FastReportExport** é necessário a instalação do componente [Fast Report](https://www.fast-report.com).

## ⚙️ Instalação Automatizada

Utilizando o [**Boss**](https://github.com/HashLoad/boss/releases/latest) (Dependency manager for Delphi) é possível instalar a biblioteca de forma automática.

```
boss install github.com/antoniojmsjr/FastReportExport
```

## ⚙️ Instalação Manual

Se você optar por instalar manualmente, basta adicionar as seguintes pastas ao seu projeto, em *Project > Options > Delphi Compiler > Target > All Configurations > Search path*

```
..\FastReportExport\Source
```

## 🧬 Provedores de Exportação

**Providers** é uma interface utilizada pela biblioteca para exportação dos relatórios e pode ser extendida para implementação de outros formatos de arquivo.

| Arquivo | Provedor |
|---|---|
| PDF | IFRExportPDF |
| HTML | IFRExportHTML |
| PNG | IFRExportPNG |
| CSV | IFRExportCSV |
| RTF | IFRExportRTF |

**Exemplo**

```delphi
var
  lFRExportPDF: IFRExportPDF;
  lFRExportHTML: IFRExportHTML;
  lFRExportPNG: IFRExportPNG;
begin

  //PROVIDER PDF
  lFRExportPDF := TFRExportProviderPDF.New;
  lFRExportPDF.frxPDF.Subject := 'Samples Fast Report Export';
  lFRExportPDF.frxPDF.Author := 'Antônio José Medeiros Schneider';

  //PROVIDER HTML
  lFRExportHTML := TFRExportProviderHTML.New;

  //PROVIDER PNG
  lFRExportPNG := TFRExportProviderPNG.New;
  lFRExportPNG.frxPNG.JPEGQuality := 100;
  
end;
```

## 🧬 DataSet de Exportação

**DataSets** é uma interface utilizada pela biblioteca para comunicação com o banco de dados através dos componentes:

| Classe | Componente |
|---|---|
| TDataSet | Nativo |
| TfrxDBDataset | Fast Report |

## ⚡️ Uso

#### Uso e definição da biblioteca

Os exemplos estão disponíveis na pasta do projeto:

```
..\FastReportExport\Samples
```

**Banco de dados de exemplo**

* Firebird: 2.5.7 [Donwload](http://sourceforge.net/projects/firebird/files/firebird-win32/2.5.7-Release/Firebird-2.5.7.27050_0_Win32.exe/download)
* Arquivo BD: 
```
..\FastReportExport\Samples\DB
```

**Relatório de exemplo**

```
..\FastReportExport\Samples\Report
```
**Exemplo**

```delphi
uses FRExport, FRExport.Types, FRExport.Interfaces.Providers;

var
  lFRExportPDF: IFRExportPDF;
  lFRExportHTML: IFRExportHTML;
  lFRExportPNG: IFRExportPNG;
  lFileStream: TFileStream;
  lFileExport: string;
begin

  //PROVIDER PDF
  lFRExportPDF := TFRExportProviderPDF.New;
  lFRExportPDF.frxPDF.Subject := 'Samples Fast Report Export';
  lFRExportPDF.frxPDF.Author := 'Antônio José Medeiros Schneider';

  //PROVIDER HTML
  lFRExportHTML := TFRExportProviderHTML.New;

  //PROVIDER PNG
  lFRExportPNG := TFRExportProviderPNG.New;
  lFRExportPNG.frxPNG.JPEGQuality := 100;

  //CLASSE DE EXPORTAÇÃO
  try
    TFRExport.New.
      DataSets.
        SetDataSet(qryCliente, 'DataSetCliente').
      &End.
      Providers.
        SetProvider(lFRExportPDF).
        SetProvider(lFRExportHTML).
        SetProvider(lFRExportPNG).
      &End.
      Export.
        SetFileReport(TUtils.PathAppFileReport). //LOCAL DO RELATÓRIO *.fr3
        Report(procedure(pfrxReport: TfrxReport) //CONFIGURAÇÃO DO COMPONENTE DE RELATÓRIO DO FAST REPORT
          var
            lfrxComponent: TfrxComponent;
            lfrxMemoView: TfrxMemoView absolute lfrxComponent;
          begin
            pfrxReport.ReportOptions.Author := 'Antônio José Medeiros Schneider';

            //PASSAGEM DE PARÂMETRO PARA O RELATÓRIO
            lfrxComponent := pfrxReport.FindObject('mmoProcess');
            if Assigned(lfrxComponent) then
            begin
              lfrxMemoView.Memo.Clear;
              lfrxMemoView.Memo.Text := 'VCL';
            end;
          end).
        Execute; //EXECUTA O PROCESSO DE EXPORTAÇÃO DO RELATÓRIO
  except
    on E: Exception do
    begin
      if E is EFRExport then
        ShowMessage(E.ToString)
      else
        ShowMessage(E.Message);
      Exit;
    end;
  end;

  //SALVAR PDF
  if Assigned(lFRExportPDF.Stream) then
  begin
    lFileStream := nil;
    try
      lFileExport := Format('%s%s', [TUtils.PathApp, 'Cliente.pdf']);
      lFileStream := TFileStream.Create(lFileExport, fmCreate);
      lFileStream.CopyFrom(lFRExportPDF.Stream, 0);
    finally
      FreeAndNil(lFileStream);
    end;
  end;

  //SALVAR HTML
  if Assigned(lFRExportHTML.Stream) then
  begin
    lFileStream := nil;
    try
      lFileExport := Format('%s%s', [TUtils.PathApp, 'Cliente.html']);
      lFileStream := TFileStream.Create(lFileExport, fmCreate);
      lFileStream.CopyFrom(lFRExportHTML.Stream, 0);
    finally
      lFileStream.Free;
    end;
  end;

  //SALVAR PNG
  if Assigned(lFRExportPNG.Stream) then
  begin
    lFileStream := nil;
    try
      lFileExport := Format('%s%s', [TUtils.PathApp, 'Cliente.png']);
      lFileStream := TFileStream.Create(lFileExport, fmCreate);
      lFileStream.CopyFrom(lFRExportPNG.Stream, 0);
    finally
      lFileStream.Free;
    end;
  end;
end;
```

**Teste de performance para aplicações web usando JMeter:**

```
..\FastReportExport\Samples\JMeter
```


https://user-images.githubusercontent.com/20980984/173268272-dc81f411-b2e5-4030-8c56-c461527f2ebc.mp4



## ⚠️ Licença
`FastReportExport` is free and open-source software licensed under the [![License](https://img.shields.io/badge/license-Apache%202-blue.svg)](https://github.com/antoniojmsjr/Horse-IPGeoLocation/blob/master/LICENSE)
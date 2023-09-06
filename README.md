# Running with ]dtest

## Test discovery

    ]dtest /Users/stefan/work/testws/tests

## Use test suite defined by a .dyalogtest file

    ]dtest /Users/stefan/work/testws/tests/mytests.dyalogtest

## Run tests against actual code

    ]link.create|import # /Users/stefan/work/testws/src

```apl
mysum ← {
    ⍺+⍵
}
```

Now write a test liks so:

```apl
test_mysum ← {
    '1+1 should equal 2'⊢2 Assert 1 #.mysum 1:
    ''
}
```

and now we can run it:

```
]dtest /Users/stefan/work/testws/tests/mytests
2023-09-03-10:23:029 *** DTest 1.84.0
    2 tests (=2 calls to "Check" or "Assert") passed in  0.0s 
```

## Running tests from the command line

Use a test runner in the same directory as the source files, e.g. `src/run.aplf`:

```apl
Run dir;testdir;results

testdir ← 'tests',⍨⊃1⎕NPARTS'[/\\]$'⎕R''⊃dir

results ← ⎕SE.UCMD'DTest ',testdir, ' -quiet'
:If 0=≢results
    ⎕←'All tests passed'
    ⎕OFF 0
:Else
    ⎕←results
    ⎕OFF 11
:EndIf
```

you can now run this in the shell like so:

```
% dyalog -b -s LOAD=src 1>&2 2>/dev/null
```

Some hoop-jumping there to get the right output sent to the shell. 

That command exits with a 0 if all went OK, and 11 otherwise.


```
docker build -t dytest .

docker run --rm \
  -v "$(pwd)/DBuildTest/DyalogBuild.dyalog:/home/dyalog/MyUCMDs/DyalogBuild.dyalog" \
  -v "$(pwd)/src:/src" \
  -v "$(pwd)/tests:/tests" \
  dytest

```

On Windows PowerShell:

    docker run --rm `
    -v "${PWD}/DBuildTest/DyalogBuild.dyalog:/home/dyalog/MyUCMDs/DyalogBuild.dyalog" `
    -v "${PWD}/src:/src" `
    -v "${PWD}/tests:/tests" `
    dytest

On Windows Command Prompt:

    docker run --rm -v "%cd%\DBuildTest\DyalogBuild.dyalog:/home/dyalog/MyUCMDs/DyalogBuild.dyalog" -v "%cd%\src:/src" -v "%cd%\tests:/tests" dytest



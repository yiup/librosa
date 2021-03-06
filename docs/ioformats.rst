Advanced I/O Use Cases
^^^^^^^^^^^^^^^^^^^^^^

This section covers advanced use cases for input and output which go beyond the I/O
functionality currently provided by *librosa*.

Read specific formats
---------------------

*librosa* uses `soundfile <https://github.com/bastibe/PySoundFile>`_ and `audioread <https://github.com/sampsyo/audioread>`_ for reading audio.
As of v0.7, librosa will use `soundfile` by default, and only fall back on `audioread` when dealing with codecs unsupported by `soundfile` (notably, MP3, and some variants of WAV).
For a list of codecs supported by `soundfile`, see the *libsndfile* `documentation <http://www.mega-nerd.com/libsndfile/>`_.

.. note:: See installation instruction for PySoundFile `here <http://pysoundfile.readthedocs.io>`_.

Librosa's load function is meant for the common case where you want to load an entire (fragment of a) recording into memory, but some applications require more flexibility.
In these cases, we recommend using `soundfile` directly.
Reading audio files using `soundfile` is similar to the method in *librosa*. One important difference is that the read data is of shape ``(nb_samples, nb_channels)`` compared to ``(nb_channels, nb_samples)`` in :func:`<librosa.core.load>`. Also the signal is not resampled to 22050 Hz by default, hence it would need be transposed and resampled for further processing in *librosa*. The following example is equivalent to ``librosa.load(librosa.util.example_audio_file())``:

.. code-block:: python
    :linenos:

    import librosa
    import soundfile as sf

    # Get example audio file
    filename = librosa.util.example_audio_file()

    data, samplerate = sf.read(filename, dtype='float32')
    data = data.T
    data_22k = librosa.resample(data, samplerate, 22050)


Blockwise Reading
-----------------

For large audio signals it could be beneficial to not load the whole audio file
into memory. *PySoundFile* supports blockwise reading. In the following example
a block of 1024 samples of audio are read and directly fed into the chroma
feature extractor.

.. code-block:: python
    :linenos:

    import numpy as np
    import soundfile as sf
    from librosa.feature import chroma_stft

    block_gen = sf.blocks('stereo_file.wav', blocksize=1024)
    rate = sf.info('stereo_file.wav').samplerate

    chromas = []
    for bl in block_gen:
        # downmix frame to mono (averaging out the channel dimension)
        y=np.mean(bl, axis=1)
        # compute chroma feature
        chromas.append(chroma_stft(y, sr=rate))



Read file-like objects
----------------------

If you want to read audio from file-like objects (also called *virtual files*)
you can use `soundfile` as well.  (This will also work with `librosa.load`, provided that the underlying codec is supported by `soundfile`.)

E.g.: read files from zip compressed archives:

.. code-block:: python
    :linenos:

    import zipfile as zf
    import soundfile as sf
    import io

    with zf.ZipFile('test.zip') as myzip:
        with myzip.open('stereo_file.wav') as myfile:
            tmp = io.BytesIO(myfile.read())
            data, samplerate = sf.read(tmp)

.. warning:: This is a example does only work in python 3. For python 2 please use ``from urllib2 import urlopen``.

Download and read from URL:

.. code-block:: python
    :linenos:

    import soundfile as sf
    import io

    from six.moves.urllib.request import urlopen

    url = "https://raw.githubusercontent.com/librosa/librosa/master/tests/data/test1_44100.wav"

    data, samplerate = sf.read(io.BytesIO(urlopen(url).read()))


Write out audio files
---------------------

*librosa* provides a thin wrapper around `scipy.io.wavfile <https://docs.scipy.org/doc/scipy/reference/generated/scipy.io.wavfile.write.html>`_ to write out WAV files. 

.. code-block:: python
    :linenos:

    import numpy as np

    rate = 44100
    data = np.random.randn(2 * rate)

    librosa.output.write_wav('file.wav', data, rate)



Please be aware that this function only supports floating-point inputs. For example if your processed audio array is of dtype ``np.float64`` (which is the default on most machines), your resulting WAV file would be of type 64-bit float as well. This is not considered to be a `standard PCM wavfile <https://msdn.microsoft.com/en-us/library/windows/hardware/dn653308%28v=vs.85%29.aspx>`_, but most WAV readers should be able to load it without problems.

Writing audio files using `PySoundFile <https://pysoundfile.readthedocs.io/en/latest/>`_ is similar to the method in librosa. However, PySoundFile can automatically convert to a given PCM subtype and additionally support several compressed formats like FLAC or OGG vorbis.

.. code-block:: python
    :linenos:

    import numpy as np
    import soundfile as sf

    rate = 44100
    data = np.random.uniform(-1, 1, size=(rate * 10, 2))

    # Write out audio as 24bit PCM WAV
    sf.write('stereo_file.wav', data, samplerate, subtype='PCM_24')

    # Write out audio as 24bit Flac
    sf.write('stereo_file.flac', data, samplerate, format='flac', subtype='PCM_24')

    # Write out audio as 16bit OGG
    sf.write('stereo_file.ogg', data, samplerate, format='ogg', subtype='vorbis')


In general, we recommend using `PySoundFile` for output rather than ``librosa.output.write_wav``.

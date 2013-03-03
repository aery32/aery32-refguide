Serial port class driver
========================

.. code-block:: c++

    #include <aery32/serial_port_clsdrv.h>

    int             putc(char c);
    int             puts(const char *str);
    int             printf(const char *format, ... );

    int             getc();
    char*           getline(char *str, [size_t *nread, char delim]);
    char*           getline(char *str, [size_t *nread, const char *delim]);

    serial_port&    putback(char c);
    serial_port&    flush();

    serial_port&    enable();
    serial_port&    disable();
    serial_port&    reset();
    
    size_t          bytes_available();
    bool            has_overflown();
    bool            is_enabled();
    
    double          set_speed(unsigned int speed);

    serial_port&    set_databits(enum Usart_databits databits);
    serial_port&    set_parity(enum Usart_parity parity);
    serial_port&    set_stopbits(enum Usart_stopbits stopbits);

    serial_port&    set_default_delim(char delim);
    serial_port&    set_default_delim(const char *delim);

    serial_port&    enable_hw_handshaking();


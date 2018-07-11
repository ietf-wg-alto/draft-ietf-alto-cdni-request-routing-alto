XML2RFC := xml2rfc --verbose -D $(shell date -u +%Y-%m-%d)
TARGET := draft-ietf-alto-cdni-request-routing-alto-04
all:
	$(XML2RFC) $(TARGET).xml

clean:
	rm -f $(TARGET).txt


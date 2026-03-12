# Automapper

## Table of content

- [Automapper](#automapper)
  - [Table of content](#table-of-content)
  - [Why would you want to use it?](#why-would-you-want-to-use-it)
  - [How to use it?](#how-to-use-it)
  - [Create custom mapping](#create-custom-mapping)
  - [Additional configurations](#additional-configurations)

Full documentation is here: <https://docs.automapper.io/en/stable/index.html>.

## Why would you want to use it?

Automapper gives you a way to map a class to another class either using common names of properites or either with a custom mapping function. You use it a lot to link DTos to models. It serves as an adapter in the adapter design pattern.

## How to use it?

Start by defining your entities and then create your mapper. For the most basic mappers, we don't need to introduce any rules because the names of the fields are the same.

```cs
using AutoMapper;

namespace CityInfo.API.Profiles
{
    public class CityProfile : Profile
    {
        public CityProfile()
        {
            CreateMap<Entities.City, Models.CityWithoutPointsOfInterestDto>();
            CreateMap<Entities.City, Models.CityDto>();
        }
    }
```

You can then add the mapper to your controller in order to do your mapping operations and convert models to DTos and vice versa.

```cs
builder.Services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());
```

```cs
using Asp.Versioning;
using AutoMapper;
using CityInfo.API.Models;
using CityInfo.API.Services;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Text.Json;

namespace CityInfo.API.Controllers
{
    [ApiController]
    [Authorize]
    [Route("api/v{version:apiVersion}/cities")]
    [ApiVersion(1)]
    [ApiVersion(2)]
    public class CitiesController : ControllerBase
    {
        private readonly ICityInfoRepository _cityInfoRepository;
        private readonly IMapper _mapper;
        const int maxCitiesPageSize = 20;

        public CitiesController(ICityInfoRepository cityInfoRepository,
            IMapper mapper)
        {
            _cityInfoRepository = cityInfoRepository ??
                throw new ArgumentNullException(nameof(cityInfoRepository));
            _mapper = mapper ??
                throw new ArgumentNullException(nameof(mapper));
        }

        [HttpGet]
        public async Task<ActionResult<IEnumerable<CityWithoutPointsOfInterestDto>>> GetCities(
                    string? name, string? searchQuery, int pageNumber = 1, int pageSize = 10)
        {
            if (pageSize > maxCitiesPageSize)
            {
                pageSize = maxCitiesPageSize;
            }

            var (cityEntities, paginationMetadata) = await _cityInfoRepository
                .GetCitiesAsync(name, searchQuery, pageNumber, pageSize);

            Response.Headers.Add("X-Pagination",
                JsonSerializer.Serialize(paginationMetadata));

            // We can map a list of elements all at once. We write the type we want to map to.
            return Ok(_mapper.Map<IEnumerable<CityWithoutPointsOfInterestDto>>(cityEntities));
        }
    }
}
```

## Create custom mapping

Imagine we have a case where the name of input and output data are not the same. We need in this case to do a custom mapping aka specify explicitely which field should map as which other field.

```cs
public class CalendarEvent
{
    public DateTime Date { get; set; }
    public string Title { get; set; }
}

public class CalendarEventForm
{
    public DateTime EventDate { get; set; }
    public int EventHour { get; set; }
    public int EventMinute { get; set; }
    public string Title { get; set; }
}

// Example of a value to map.
var calendarEvent = new CalendarEvent
{
    Date = new DateTime(2008, 12, 15, 20, 30, 0),
    Title = "Company Holiday Party"
};

// Configure AutoMapper
var configuration = new MapperConfiguration(cfg =>
  cfg.CreateMap<CalendarEvent, CalendarEventForm>()
    .ForMember(dest => dest.EventDate, opt => opt.MapFrom(src => src.Date.Date))
    .ForMember(dest => dest.EventHour, opt => opt.MapFrom(src => src.Date.Hour))
    .ForMember(dest => dest.EventMinute, opt => opt.MapFrom(src => src.Date.Minute)), loggerFactory);

// Perform mapping
CalendarEventForm form = mapper.Map<CalendarEvent, CalendarEventForm>(calendarEvent);

// Tests to verify that our mapping took place successfully.
form.EventDate.ShouldEqual(new DateTime(2008, 12, 15));
form.EventHour.ShouldEqual(20);
form.EventMinute.ShouldEqual(30);
form.Title.ShouldEqual("Company Holiday Party");
```

## Additional configurations

When nothing is specified, automapper will map only if the names of the elements are absolutely equal. In order to have a bit more, you can add a mapperCOngiguration to specify how mapping should be done. You have an example under where we have a special case and you will need a little help in the mapping.

```cs
public class Source
{
    public int Value { get; set; }
    public int Ävíator { get; set; }
    public int SubAirlinaFlight { get; set; }
}
public class Destination
{
    public int Value { get; set; }
    public int Aviator { get; set; }
    public int SubAirlineFlight { get; set; }
}

var configuration = new MapperConfiguration(c =>
{
    c.ReplaceMemberName("Ä", "A");
    c.ReplaceMemberName("í", "i");
    c.ReplaceMemberName("Airlina", "Airline");
}, loggerFactory);
```

There are also ways to work with prefix and suffix to simplify mapping.

```cs
public class Source {
    public int frmValue { get; set; }
    public int frmValue2 { get; set; }
}
public class Dest {
    public int Value { get; set; }
    public int Value2 { get; set; }
}
var configuration = new MapperConfiguration(cfg => {
    cfg.RecognizePrefixes("frm");
    cfg.CreateMap<Source, Dest>();
}, loggerFactory);

configuration.AssertConfigurationIsValid(); // Creates an error if the configuration is invalid.
```
